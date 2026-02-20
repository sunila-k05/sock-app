pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Docker-Compose Pull') {
            steps {
                echo "Pulling base images..."
                sh 'docker-compose pull'
            }
        }

        stage('Docker-Compose Build') {
            steps {
                echo "Building services..."
                sh 'docker-compose build'
            }
        }

        stage('Docker-Compose Up') {
            steps {
                echo "Starting containers..."
                sh 'docker-compose up -d'
            }
        }

        stage('Verify Containers') {
            steps {
                echo "Verifying running containers..."
                sh 'docker ps'
            }
        }
    }

    post {

        always {
            echo "Stopping and cleaning containers..."
            sh 'docker-compose down'
        }

        success {
            echo "CI validation completed successfully."
        }

        failure {
            echo "CI validation failed."
        }
    }
}
