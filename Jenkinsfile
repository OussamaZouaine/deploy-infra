pipeline {
    agent any

    options {
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
        always {
            echo '🧹 cleaning up the workspace...'
            deleteDir()
        }
        success {
            echo '✅ Infra Stack started successfully. frontend, backend, and database are running.'
            slackSend(
                color: 'good',
                message: "✅ *${env.JOB_NAME}* #${env.BUILD_NUMBER} succeeded\n<${env.BUILD_URL}|Open build> • Branch: ${env.BRANCH_NAME ?: 'N/A'} • Image: `${env.IMAGE_NAME}:${env.IMAGE_TAG ?: 'n/a'}`"
            )
        }
        failure {
            echo '❌ Failed to run Docker Compose. Check logs above.'
            slackSend(
                color: 'danger',
                message: "❌ *${env.JOB_NAME}* #${env.BUILD_NUMBER} failed\n<${env.BUILD_URL}|Open build> • Branch: ${env.BRANCH_NAME ?: 'N/A'}"
            )
        }
        unstable {
            slackSend(
                color: 'warning',
                message: "⚠️ *${env.JOB_NAME}* #${env.BUILD_NUMBER} is unstable\n<${env.BUILD_URL}|Open build>"
            )
        }
    }
}
