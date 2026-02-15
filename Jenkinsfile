pipeline {
    agent any

    options {
        skipStaleStageBuild()
        timeout(time: 15, unit: 'MINUTES')
    }

    environment {
        COMPOSE_PROJECT_NAME = 'deploy-infra'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    // Use 'docker compose' (V2) or 'docker-compose' (V1) depending on your Jenkins agent
                    sh 'docker compose pull --quiet'
                    sh 'docker compose up -d'
                }
            }
        }
    }

    post {
        success {
            echo 'Stack started successfully. frontend, backend, and database are running.'
        }
        failure {
            echo 'Failed to run Docker Compose. Check logs above.'
        }
        cleanup {
            // Optional: uncomment to tear down the stack when the job ends
            // sh 'docker compose down || true'
        }
    }
}
