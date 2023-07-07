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
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/db-map.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/pv.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/pvc.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/db-stateset.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/db-svc.yaml'
      }
    }
    stage('Fetching Database') {
      steps {
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-jenkins/fetch-job.yaml'
        }
      }
    
    stage('Storing artifacts') {
      steps {
        script{
            sh 'sudo kubectl --kubeconfig=${KUBECONFIG} cp fetch-db-pod:/mnt/database.sql database.sql'
            archiveArtifacts artifacts: 'database.sql' 
        }
      }
    }
  }
}  
