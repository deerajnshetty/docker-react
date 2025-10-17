pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-repo')
        IMAGE_NAME = " deeraj1986/react-app"
        DEPLOY_SERVER = "ubuntu@47.129.51.50"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/deerajnshetty/docker-react.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME:latest .'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
              sshagent (credentials: ['ec2-ssh-key']) {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER '
                    docker pull $IMAGE_NAME:latest &&
                    docker stop react-app || true &&
                    docker rm react-app || true &&
                    docker run -d --name react-app -p 80:80 $IMAGE_NAME:latest
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
