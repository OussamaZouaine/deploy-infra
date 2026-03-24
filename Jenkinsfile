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
                    sh '''
                        set -e
                        if docker compose version >/dev/null 2>&1; then
                            docker compose pull
                            docker compose up -d
                        elif command -v docker-compose >/dev/null 2>&1; then
                            docker-compose pull
                            docker-compose up -d
                        else
                            echo "Neither Docker Compose V2 (docker compose) nor docker-compose (v1) is installed on this agent." >&2
                            exit 1
                        fi
                    '''
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
