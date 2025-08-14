pipeline {
  agent {
    docker {
        image 'kalra1994/helm-kubectl-docker:latest'
        args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  environment {
    DOCKERHUB_REPO = "kalra1994"
  }
  stages {
    stage('Clone Repos') {
      steps {
        checkout scm
        dir('learnerReportCS_frontend') {
            git url: 'https://github.com/UnpredictablePrashant/learnerReportCS_frontend', branch: 'main'
        }
        dir('learnerReportCS_backend') {
            git url: 'https://github.com/UnpredictablePrashant/learnerReportCS_backend', branch: 'main'
        }
      }
    }
    stage('Build Docker Images') {
      steps {
        sh 'docker build -t $DOCKERHUB_REPO/learnerreportcs-frontend:latest ./learnerReportCS_frontend'
        sh 'docker build -t $DOCKERHUB_REPO/learnerreportcs-backend:latest ./learnerReportCS_backend'
      }
    }
    stage('Push Docker Images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
          sh 'docker push $DOCKERHUB_REPO/learnerreportcs-frontend:latest'
          sh 'docker push $DOCKERHUB_REPO/learnerreportcs-backend:latest'
        }
      }
    }
    stage('Deploy to K8s via Helm') {
      steps {
        sh 'kubectl apply -f k8s/mongo-secret.yaml'
        sh 'helm upgrade --install mern-app ./charts/mern-app --values ./charts/mern-app/values.yaml'
      }
    }
  }
}
