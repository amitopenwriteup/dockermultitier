pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_VERSION = '3.8'
        DB_IMAGE = 'mariadb:10-focal'
        MYSQL_ROOT_PASSWORD = readFile('./secrets/db-password').trim()
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from your GitHub repository
                git branch: 'main', url: 'https://github.com/amitopenwriteup/dockercomposetheetier.git'
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    // Build backend service
                    sh 'docker compose -f docker-compose.yaml build backend'
                }
            }
        }

        stage('Start Services') {
            steps {
                script {
                    // Start all services (db, backend, proxy)
                    sh 'docker compose -f docker-compose.yaml up -d'
                }
            }
        }

        stage('Wait for DB Service') {
            steps {
                script {
                    // Wait for DB service to become healthy
                    sh 'while ! docker compose -f docker-compose.yaml exec db mysqladmin ping -h 127.0.0.1 --silent --password="$MYSQL_ROOT_PASSWORD"; do sleep 5; done'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Deploying proxy service
                    sh 'docker compose -f docker-compose.yaml up -d proxy'
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up containers and volumes
                sh 'docker compose -f docker-compose.yaml down --volumes'
            }
        }
        success {
            echo 'Build and Deployment successful!'
        }
        failure {
            echo 'Build or Deployment failed.'
        }
    }
}
