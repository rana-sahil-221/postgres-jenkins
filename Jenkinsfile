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
          git branch: 'main',url: 'https://github.com/rana-sahil-221/postgres-jenkins.git'
    }
    }
    stage('Deploying Manifests to the K8 Cluster') {
      steps {
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-k8/db-map.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-k8/db-stateset.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-k8/db-svc.yaml'
      }
    }
    stage('Fetching Database and storing artifacts') {
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
      waitForQualityGate abortPipeline: true
}
}
