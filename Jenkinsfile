pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = "root123"
    }

    options {
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
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

        stage('Trivy Security Scan') {
            steps {
                sh '''
                echo "Scanning built images with Trivy..."
                IMAGES=$(docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>")

                for image in $IMAGES; do
                    echo "Scanning $image"
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $image
                done
                '''
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
                sh '''
                echo "Running Containers:"
                docker ps
                '''
            }
        }
    }

    post {
        always {
            dir('deploy/docker-compose') {
                sh 'docker-compose down || true'
            }
        }

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed. Check above logs."
        }
    }
}
