pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "570220875158"
        ECR_REPOSITORY = "jenkinsfile_task"
        IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/harisabbas/PHP_Application-.git'
            }
        }

        stage('Pull PHP Image') {
            steps {
                sh '''
                    docker pull php:8.3-apache
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t php-app .
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f php-container || true

                    docker run -d \
                        --name php-container \
                        -p 8081:80 \
                        php-app
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://localhost:8081
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                        --username AWS \
                        --password-stdin \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag php-app:latest $IMAGE_URI
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $IMAGE_URI
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rm -f php-container || true
            '''
        }
    }
}
