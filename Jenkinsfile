pipeline {
    agent any

    environment {
        AWS_REGION   = "us-east-1"
        ECR_REGISTRY = "173471018017.dkr.ecr.us-east-1.amazonaws.com"
        ECR_REPO     = "${ECR_REGISTRY}/income-expense-app"
        CLUSTER_NAME = "income-expense-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Madhusudhanhub/incomeExpense.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                // Use single quotes so shell variables are passed correctly
                sh '''
                # Authenticate Docker to ECR
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY

                # Build and tag image
                docker build -t income-expense-app .

                # Tag with full ECR repo path
                docker tag income-expense-app:latest $ECR_REPO:latest

                # Push to ECR
                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                # Update kubeconfig to talk to the cluster
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                # Update deployment image
                kubectl set image deployment/income-expense-deployment income-expense-container=$ECR_REPO:latest

                # Wait for rollout to finish
                kubectl rollout status deployment/income-expense-deployment
                '''
            }
        }
    }
}

