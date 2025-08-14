
pipeline {
  agent {
    docker {
      image 'kalra1994/helm-kubectl-docker:latest'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    DOCKERHUB_REPO = "kalra1994"
    DOCKERHUB_CREDENTIALS_ID = 'dockerhub-creds'
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
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
        // Fix the frontend Dockerfile to use legacy peer dependencies
        script {
          dir('learnerReportCS_frontend') {
              echo "Fixing frontend Dockerfile for build..."
              sh "sed -i 's/RUN npm install --silent/RUN npm install --legacy-peer-deps/g' Dockerfile"
          }
        }
        sh "docker build -t ${env.DOCKERHUB_REPO}/learnerreportcs-frontend:latest ./learnerReportCS_frontend"
        sh "docker build -t ${env.DOCKERHUB_REPO}/learnerreportcs-backend:latest ./learnerReportCS_backend"
      }
    }

    stage('Push Docker Images') {
      steps {
        withEnv(["HOME=${env.WORKSPACE}"]) {
          withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
              sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
              sh "docker push ${env.DOCKERHUB_REPO}/learnerreportcs-frontend:latest"
              sh "docker push ${env.DOCKERHUB_REPO}/learnerreportcs-backend:latest"
          }
        }
      }
    }

    stage('Deploy to K8s via Helm') {
      steps {
        withKubeConfig([credentialsId: KUBECONFIG_CREDENTIAL_ID]) {
            sh 'kubectl apply -f k8s/mongo-secret.yaml'
            sh 'helm upgrade --install mern-app ./charts/mern-app --values ./charts/mern-app/values.yaml'
        }
      }
    }
  }
}
