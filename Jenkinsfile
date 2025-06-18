pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '977098985978'
        AWS_REGION = 'us-east-1'
        ECR_REPO_NAME = 'flask-aws-app'
        IMAGE_TAG = 'latest'
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        APP_DIR = 'flask-aws-app'
    }

    options {
        skipDefaultCheckout()
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        stage('Clone App Repo') {
    steps {
        sh '''
        rm -rf flask-aws-app
        git clone https://github.com/Gambitdevops/flask-aws-app.git
        '''
    }
}
        stage('Build Docker Image') {
            steps {
                dir('flask-aws-app') {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@54.208.192.244 << EOF
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                docker pull $DOCKER_IMAGE
                docker stop flask-app || true && docker rm flask-app || true
                docker run -d --name flask-app -p 80:5000 $DOCKER_IMAGE
                EOF
                '''
            }
        }
    }
}
