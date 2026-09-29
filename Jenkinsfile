pipeline {

    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'devops-appgestiondesprojets'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh 'docker compose down || true'
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Verify Containers') {
            steps {
                sh 'docker compose ps'
            }
        }

        stage('Test Backend') {
            steps {
                sh '''
                    echo "Waiting for backend..."

                    for i in $(seq 1 30); do

                        if curl -f http://localhost:8081/entreprise/all; then
                            echo "Backend is running successfully!"
                            exit 0
                        fi

                        echo "Backend not ready yet... attempt $i/30"
                        sleep 3
                    done

                    echo "Backend failed to start."
                    exit 1
                '''
            }
        }

        stage('Test Frontend') {
            steps {
                sh '''
                    echo "Waiting for frontend..."

                    for i in $(seq 1 20); do

                        if curl -f http://localhost:4200; then
                            echo "Frontend is running successfully!"
                            exit 0
                        fi

                        echo "Frontend not ready yet... attempt $i/20"
                        sleep 2
                    done

                    echo "Frontend failed to start."
                    exit 1
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Pipeline failed!'

            sh '''
                echo "===== Docker Compose Status ====="
                docker compose ps || true

                echo "===== Docker Compose Logs ====="
                docker compose logs --no-color --tail=100 || true
            '''
        }

        always {
            sh 'docker compose ps || true'
        }
    }
}
