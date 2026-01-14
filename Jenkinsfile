pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "173471018017.dkr.ecr.us-east-1.amazonaws.com/income-expense-app"
        CLUSTER_NAME = "income-expense-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/incomeExpense.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                $(aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO)
                docker build -t income-expense-app .
                docker tag income-expense-app:latest $ECR_REPO:latest
                docker push $ECR_REPO:latest
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                kubectl set image deployment/income-expense-deployment income-expense-container=$ECR_REPO:latest
                kubectl rollout status deployment/income-expense-deployment
                """
            }
        }
    }
}

