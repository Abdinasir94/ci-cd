pipeline {
    agent any
    environment {
        AWS_REGION = 'eu-west-1'            // AWS region
        AWS_ACCOUNT_ID = '790886830806' // Replace with your actual AWS Account ID
        ECR_REPO_NAME = 'ci-cd'             // ECR Repository Name
        IMAGE_TAG = 'latest'                // Docker image tag
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'a0560c70-c70c-4ca5-94b2-845c30810b4',
                    url: 'https://github.com/Abdinasir94/ci-cd.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: '354df2ca-2714-428a-bdbb-5be2216d180e',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region ${AWS_REGION}
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    docker.image("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}").push()
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['6a554f5a-7c8f-42e0-9109-6c8ad9ae9c']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<target-ec2-ip> << EOF
                    docker pull ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                    docker stop ci-cd || true
                    docker rm ci-cd || true
                    docker run -d --name ci-cd -p 3000:3000 ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                    EOF
                    '''
                }
            }
        }
    }
}
