
pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }


     stage ('docker-compose pull'){
         steps {
               sh 'docker compose pull'

}

}

stage ('docker-compose up'){
     steps{
sh 'docker-compose up'
}
}

stage ('Verify Containers are running'){
steps{
sh ' docker ps'
}
}

post {
always {
sh 'docker-compose down'
}

success {
    echo "compose validation successfull"
}

failure {
echo "compose validation failed"
}
}

     stage('Verify') {
            steps {
                echo "Repository cloned successfully!"
                sh 'ls -la'
            }
        }




}

}
