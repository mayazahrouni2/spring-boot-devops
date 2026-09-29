pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test Maven') {
            steps {
                sh './mvnw clean test package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t backend-app:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-registry-credentials',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$REGISTRY_PASSWORD" | docker login localhost:5000 \
                            -u "$REGISTRY_USER" \
                            --password-stdin

                        docker tag backend-app:latest localhost:5000/backend-app:latest

                        docker push localhost:5000/backend-app:latest

                        docker logout localhost:5000
                    '''
                }
            }
        }

    }
}
