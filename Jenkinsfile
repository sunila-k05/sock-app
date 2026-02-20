pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 40, unit: 'MINUTES')
    }

    environment {
        MYSQL_ROOT_PASSWORD = "root123"
        NAMESPACE = "sock-shop"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ---------- LOCAL BUILD ----------

        stage('Docker-Compose Build') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose build'
                }
            }
        }

        // ---------- SECURITY SCAN ----------

        stage('Trivy Scan') {
            steps {
                sh '''
                echo "Scanning Docker images with Trivy..."

                IMAGES=$(docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>")

                for image in $IMAGES; do
                    echo "Scanning $image"
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $image
                done
                '''
            }
        }

        // ---------- OPTIONAL LOCAL RUN ----------

        stage('Docker-Compose Up (Local Test)') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose up -d'
                }
            }
        }

        stage('Verify Local Containers') {
            steps {
                sh 'docker ps'
            }
        }

        stage('Stop Local Containers') {
            steps {
                dir('deploy/docker-compose') {
                    sh 'docker-compose down || true'
                }
            }
        }

        // ---------- KUBERNETES DEPLOY ----------

        stage('Validate Kubernetes Manifests') {
            steps {
                sh '''
                kubectl apply --dry-run=client -f deploy/kubernetes/manifests/
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deploy/kubernetes/manifests/
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/front-end -n $NAMESPACE
                '''
            }
        }

        stage('Verify Pods') {
            steps {
                sh '''
                kubectl get pods -n $NAMESPACE
                '''
            }
        }
    }
}
