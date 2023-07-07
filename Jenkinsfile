pipeline {
  environment {
    KUBECONFIG = credentials('kube_vagrant_id')
  }
  agent {
        label 'vagrant-vm'
    }
  stages {
    stage('Cloning Repo') {
      steps {
        git branch:'main',url: 'https://github.com/rana-sahil-221/postgres-jenkins.git'
      }
    }
    stage('Deploying Manifests to the K8 Cluster') {
      steps {
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/db-map.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/pv.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/pvc.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/db-stateset.yaml'
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/db-svc.yaml'
      }
    }
    stage('Fetching Database') {
      steps {
        sh 'sudo kubectl --kubeconfig=${KUBECONFIG} apply -f /var/lib/jenkins/workspace/postgres-vagrant/fetch-job.yaml'
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
