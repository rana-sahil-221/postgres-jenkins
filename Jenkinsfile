pipeline {
  environment {
    KUBECONFIG = credentials('kube_id')
    SCANNER_HOME = tool 'sonarscanner'
    ORGANIZATION = "rana-sahil-221"
    PROJECT_NAME = "rana-sahil-221_postgres-jenkins"
  }
  agent any
  stages {
    stage('Cloning Repo') {
      steps {
          //git branch: '${branch_name}' ,url: 'https://github.com/rana-sahil-221/postgres-jenkins.git'
        checkout scmGit(branches: [[name: '${branch_name}']], extensions: [], userRemoteConfigs: [[credentialsId: '9624a2a7-70af-4b64-9eca-892f819707cb', url: 'https://github.com/rana-sahil-221/postgres-jenkins.git']])
    }
    }
    stage('Deploying Manifests to the K8 Cluster') {
      steps {
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f ${WORKSPACE}/db-map.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f ${WORKSPACE}/db-stateset.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f ${WORKSPACE}/db-svc.yaml'
      }
    }
    stage('Fetching Database with User Input') {
      steps {
          script{
            def dbList=sh(script:"sudo kubectl --kubeconfig=${KUBECONFIG} exec postgresdb-0 -- psql -h localhost -U sahil -p 5432 -c 'SELECT datname FROM pg_database' -t", returnStdout:true).trim().split('\n')
            echo "Available Databases:"
            echo dbList.join('\n')

            def userInput=input(message: 'Enter the database name: ',parameters: [string(name: 'databaseName', defaultValue: '', description: 'Name of the database to fetch')])
            sh "sudo kubectl --kubeconfig=${KUBECONFIG} exec postgresdb-0 -- pg_dump -h localhost -U sahil ${userInput} > dbArtifact.sql"
          }
        }
      }
    
    stage('Storing Artifacts') {
      steps {
        script {
          archiveArtifacts artifacts: 'dbArtifact.sql'
        }
      }
    }

   stage('SonarQube Analysis') {
     steps {
     withSonarQubeEnv('sonarserver1') {
       sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.organization=$ORGANIZATION \
        -Dsonar.java.binaries=build/classes/java/ \
        -Dsonar.projectKey=$PROJECT_NAME \
        -Dsonar.sources=.'''
    }
    }
  }

  stage('Quality Gates') {
    steps {
      timeout(time: 1, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
}
}
  }
  }

   post {
        success {
          script {
                def changeSet = currentBuild.changeSets
                def commitMsg = "No Commits"

                if (changeSet != null && changeSet.size() > 0) {
                    commitMsg = changeSet[0].items[0].msg
                }
          }
                slackSend color: "good", message: "Deployment to K8 cluster done and artifact stored!", attachments: [[
                    color: 'good',
                    title: "BUILD DETAILS",
                    fields: [
                        [
                            title: "User",
                            value: "${env.BUILD_USER}",
                            short: true
                        ],
                        [
                            title: "BUILD NUMBER",
                            value: "${currentBuild.number}",
                            short: true
                        ],
                        [
                            title: "Changelog",
                            value: commitMsg,
                            color: "good"
                        ],
                        [
                            title: "JOB URL",
                            value: "${env.JOB_URL}",
                            short: true
                        ]
                    ]
                  ]]                                                                                                
            }
      
  failure {
      slackSend (color: "danger", message: "Deployment to K8 cluster failed!", attachments: [[
        color: 'danger',
        title: "BUILD DETAILS",
        fields: [[
          title: "User",
          value: "${env.BUILD_USER}",
          short: true
        ],
        [
          title: "BUILD NUMBER",
          value: "${currentBuild.number}",
          short: true
        ],
        [
          title: "JOB URL",
          value: "${env.JOB_URL}",
          short: true
        ]]
      ]]
    )
  }
  }
}
//jenkinsfile of branch-1
