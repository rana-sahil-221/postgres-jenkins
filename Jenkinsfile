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
            def dbList=sh(script:"kubectl --kubeconfig=${KUBECONFIG} exec postgresdb-0 -- psql -h 192.168.56.10 -U sahil -p 5432 -c 'SELECT datname FROM pg_database' -t", returnStdout:true).trim().split('\n')
            echo "Available Databases:"
            echo dbList.join('\n')

            def userInput=input(message: 'Enter the database name: ',parameters: [string(name: 'databaseName', defaultValue: '', description: 'Name of the database to fetch')])
            sh "kubectl --kubeconfig=${KUBECONFIG} exec postgresdb-0 -- pg_dump -h 192.168.56.10 -U sahil ${userInput} > database.sql"
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
