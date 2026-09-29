pipeline {
    agent any

    stages {

        stage('Build Maven') {
            steps {
                sh './mvnw clean package -DskipTests'
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

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker network create devops-network 2>/dev/null || true

                    docker rm -f mysql 2>/dev/null || true

                    docker run -d \
                        --name mysql \
                        --network devops-network \
                        -e MYSQL_ROOT_PASSWORD=root \
                        -e MYSQL_DATABASE=test_db \
                        mysql:8.0

                    echo "Waiting for MySQL..."

                    until docker exec mysql \
                        mysqladmin ping -h localhost -uroot -proot --silent
                    do
                        sleep 2
                    done

                    echo "MySQL is ready."
                '''
            }
        }

        stage('Deploy Backend') {
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

                        docker rm -f backend-app 2>/dev/null || true

                        docker pull localhost:5000/backend-app:latest

                        docker run -d \
                            --name backend-app \
                            --network devops-network \
                            -p 8080:8080 \
                            localhost:5000/backend-app:latest

                        docker logout localhost:5000
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== Docker Containers ====="
                    docker ps

                    echo "===== Backend Logs ====="
                    docker logs backend-app
                '''
            }
        }
    }

    post {
        success {
            echo '=========================================='
            echo '     DEPLOYMENT SUCCESSFUL'
            echo '=========================================='
        }

        failure {
            echo '=========================================='
            echo '       PIPELINE FAILED'
            echo '=========================================='
        }
    }
}
