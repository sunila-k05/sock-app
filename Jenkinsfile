                                                        
pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = "root123"
    }

    options {
        timestamps()
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker-Compose Pull') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose pull'
                }
            }
        }

        stage('Docker-Compose Build') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose build'
                }
            }
        }

        stage('Docker-Compose Up') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose up -d'
                }
            }
        }

        stage('Verify Containers') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        always {
            dir('deploy/docker-compose') {
                sh 'docker-compose down || true'
            }
        }
    }
}
