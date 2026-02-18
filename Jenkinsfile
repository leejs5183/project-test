pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-northeast-2'
    ECR_REGISTRY = '579378699580.dkr.ecr.ap-northeast-2.amazonaws.com'
    ECR_REPO = 'my-app'
    IMAGE_TAG = "${BUILD_NUMBER}"
    IMAGE_URI = "${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
  }

  stages {

    stage('Build WAR') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t my-app:${BUILD_NUMBER} .'
      }
    }

    stage('ECR Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'aws-creds',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
        }
      }
    }

    stage('Push to ECR') {
      steps {
        sh 'docker tag my-app:${BUILD_NUMBER} ${IMAGE_URI}'
        sh 'docker push ${IMAGE_URI}'
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh 'kubectl set image deployment/my-app my-app=${IMAGE_URI}'
        sh 'kubectl rollout status deployment/my-app'
      }
    }

  }
}

