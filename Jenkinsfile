pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'a0560c70-c70c-4ca5-94b2-845c30810b4',  // GitHub credentials ID
                    url: 'https://github.com/Abdinasir94/ci-cd.git'        // Repository URL
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Change directory to the folder containing the Dockerfile
                    dir('sample-node-app') {
                        // Build the Docker image and tag it for ECR
                        dockerImage = docker.build("790886830806.dkr.ecr.eu-west-1.amazonaws.com/ci-cd:latest")
                    }
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                sh '''
                    # Hardcoded AWS credentials (replace YOUR_AWS_ACCESS_KEY and YOUR_AWS_SECRET_KEY with actual values)
                    aws configure set aws_access_key_id AKIA3QJEHKLLAEOGE2WM
                    aws configure set aws_secret_access_key H9a/1uuSPsVK0N1t2chcXyyoNJJYeK8rKfS9ij7e
                    aws configure set default.region eu-west-1
                    # Login to AWS ECR
                    aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 790886830806.dkr.ecr.eu-west-1.amazonaws.com
                '''
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the Docker image to the ECR repository
                    dockerImage.push()
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['6a554f5a-7c8f-42e0-9109-6c8ad9ae9c']) {  // SSH credentials for EC2
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@52.31.190.71 << EOF
                    # Pull the latest Docker image from ECR
                    docker pull 790886830806.dkr.ecr.eu-west-1.amazonaws.com/ci-cd:latest
                    # Stop and remove any existing container
                    docker stop ci-cd || true
                    docker rm ci-cd || true
                    # Run the new container on port 3000
                    docker run -d --name ci-cd -p 3000:3000 790886830806.dkr.ecr.eu-west-1.amazonaws.com/ci-cd:latest
                    EOF
                    '''
                }
            }
        }
    }
}
