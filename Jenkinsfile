pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
    }

    environment {
        STAGE_NS = "sock-stage"
        PROD_NS  = "sock-prod"
        RELEASE  = "sockshop"
        CHART_PATH = "deploy/kubernetes/helm-chart"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Trivy Scan Images') {
            steps {
                sh '''
                echo "Scanning images..."
                IMAGES=$(docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>")
                for image in $IMAGES; do
                    trivy image --exit-code 1 --severity HIGH,CRITICAL $image
                done
                '''
            }
        }

        // ---------------- STAGING ----------------

        stage('Deploy to Staging') {
            steps {
                sh """
                helm upgrade --install $RELEASE $CHART_PATH \
                  --namespace $STAGE_NS \
                  --create-namespace \
                  -f $CHART_PATH/values-stage.yaml
                """
            }
        }

        stage('Verify Staging Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/front-end -n $STAGE_NS
                '''
            }
        }

        // ---------------- APPROVAL ----------------

        stage('Approval for Production') {
            steps {
                input message: "Approve deployment to Production?"
            }
        }

        // ---------------- PRODUCTION ----------------

        stage('Deploy to Production') {
            steps {
                sh """
                helm upgrade --install $RELEASE $CHART_PATH \
                  --namespace $PROD_NS \
                  --create-namespace \
                  -f $CHART_PATH/values-prod.yaml
                """
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
            script {
                echo "Deployment failed. Rolling back..."
                sh """
                helm rollback $RELEASE 1 -n $PROD_NS || true
                """
            }
        }
    }
}
