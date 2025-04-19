pipeline {
  agent any

  environment {
    ECR_REPO = '890742580558.dkr.ecr.ap-south-1.amazonaws.com/node-app'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    IMAGE_URI = "${ECR_REPO}:${IMAGE_TAG}"
    AWS_REGION = 'ap-south-1'
    GITHUB_TOKEN = credentials('github-token')
  }

  stages {
    stage('Checkout') {
      steps {
        git url:'https://amalspillai02:${GITHUB_TOKEN}@github.com/amalspillai02/project3.git', branch:'main'
      }
    }

    stage('Install & Test') {
      steps {
        sh 'npm install'
        sh 'npm test'  // optional
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE_URI} ."
      }
    }

    stage('Login to ECR') {
      steps {
        sh """
          aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
        """
      }
    }

    stage('Push Docker Image') {
      steps {
        sh "docker push ${IMAGE_URI}"
      }
    }

    stage('Update Rollout YAML') {
      steps {
        sh "sed -i 's|image: .*|image: ${IMAGE_URI}|' rollout.yaml"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh "kubectl apply -f rollout.yaml"
      }
    }
  }
}
