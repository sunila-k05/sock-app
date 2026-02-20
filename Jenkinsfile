pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        RELEASE = "sockshop"
        CHART_PATH = "deploy/kubernetes/helm-chart"
        STAGE_NS = "sock-stage"
        PROD_NS  = "sock-prod"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                helm upgrade --install $RELEASE $CHART_PATH \
                  --namespace $STAGE_NS \
                  --create-namespace
                '''
            }
        }

        stage('Verify Staging Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/front-end -n $STAGE_NS
                '''
            }
        }

        stage('Approve Production Deployment') {
            steps {
                input "Deploy to Production?"
            }
        }

        stage('Deploy to Production') {
            steps {
                sh '''
                helm upgrade --install $RELEASE $CHART_PATH \
                  --namespace $PROD_NS \
                  --create-namespace
                '''
            }
        }

        stage('Verify Production Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/front-end -n $PROD_NS
                '''
            }
        }
    }

    post {
        failure {
            echo "Deployment failed. Rolling back..."
            sh '''
            helm rollback $RELEASE 1 -n $PROD_NS || true
            '''
        }
    }
}
