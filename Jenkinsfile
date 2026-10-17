pipeline {
    agent any

    environment {
        COMPOSE_PROJECT = 'devops-appgestiondesprojets'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t gestion-projets-backend:${BUILD_NUMBER} -t gestion-projets-backend:latest ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t gestion-projets-frontend:${BUILD_NUMBER} -t gestion-projets-frontend:latest ./frontend'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose -p ${COMPOSE_PROJECT} up -d --build'
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                for i in $(seq 1 30); do
                  if docker run --rm --network ${COMPOSE_PROJECT}_default curlimages/curl:latest -sf http://backend:8080/entreprise/all > /dev/null; then
                    echo "Backend OK"
                    docker run --rm --network ${COMPOSE_PROJECT}_default curlimages/curl:latest -sf http://frontend/ > /dev/null && echo "Frontend OK"
                    exit 0
                  fi
                  echo "Attente du backend ($i/30)..."
                  sleep 5
                done
                echo "Backend KO"
                exit 1
                '''
            }
        }
    }

    post {
        success { echo 'Pipeline terminé avec succès' }
        failure { echo 'Pipeline en échec : voir les logs' }
    }
}