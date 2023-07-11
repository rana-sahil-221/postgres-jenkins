pipeline {
  environment {
    KUBECONFIG = credentials('kube_id')
  }
  agent any
  stages {
    stage('Cloning Repo') {
      steps {
        git branch:'main',url: 'https://github.com/rana-sahil-221/postgres-jenkins.git'
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
            def dbList=sh(script:"sudo kubectl --kubeconfig=${KUBECONFIG} exec postgresdb-0 -- /bash PGPASSWORD=Sahil@123 psql -h localhost -U sahil -c 'SELECT datname FROM pg_database' -t", returnStdout:true).trim().split('\n')
            echo "Available Databases:"
            echo dbList.join('\n')

            def userInput=input(message: 'Enter the database name: ',parameters: [string(name: 'databaseName', defaultValue: '', description: 'Name of the database to fetch')])
            sh "PGPASSWORD=Sahil@123 pg_dump -h localhost -U sahil ${userInput} > database.sql"
          }
        }
      }
    
    stage('Storing Artifacts') {
      steps {
        script {
          archiveArtifacts artifacts: 'database.sql'
        }
      }
    }
  }
}  
