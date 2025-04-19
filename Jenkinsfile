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
        sh 'ls -l'  // List files to confirm the file path
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
        withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh """
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region ${AWS_REGION}
            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
          """
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        sh "docker push ${IMAGE_URI}"
      }
    }

    stage('Authenticate with EKS') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh """
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region ${AWS_REGION}
            aws eks --region ${AWS_REGION} update-kubeconfig --name project-cluster
          """
        }
      }
    }

    stage('Update Rollout YAML') {
      steps {
        // Modify the rollout.yaml file in the k8s folder
        sh "sed -i 's|image: .*|image: ${IMAGE_URI}|' k8s/rollout.yaml"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        // Apply the updated YAML file to Kubernetes
        sh "kubectl apply -f k8s/rollout.yaml"
      }
    }
  }
}
