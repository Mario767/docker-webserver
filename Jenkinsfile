pipeline {
    agent any
    environment {
        APP_SERVER = '54.152.219.229'
        APP_USER = 'ec2-user'
        IMAGE_NAME = 'mario-webserver'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Mario767/docker-webserver.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t mario-webserver:latest .'
            }
        }
        stage('Test') {
            steps {
                sh '''
                    docker run -d --name test-container -p 8081:80 mario-webserver:latest
                    sleep 3
                    curl -f http://localhost:8081 | grep "Mario Lucić"
                    docker stop test-container
                    docker rm test-container
                '''
            }
        }
        stage('Deploy') {
            steps {
                sshagent(['app-server-key']) {
                    sh '''
                        docker save mario-webserver:latest | ssh -o StrictHostKeyChecking=no ec2-user@54.152.219.229 'docker load'
                        scp -o StrictHostKeyChecking=no docker-compose.yml ec2-user@54.152.219.229:/home/ec2-user/
                        ssh -o StrictHostKeyChecking=no ec2-user@54.152.219.229 'docker-compose down && docker-compose up -d'
                    '''
                }
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed! Deploy se nece izvrsiti.'
        }
        success {
            echo 'Deploy uspjesan! Web server dostupan na http://54.152.219.229'
        }
    }
}
