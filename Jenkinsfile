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
                        COMPOSE_VERSION="v2.29.2"
                        if docker compose version >/dev/null 2>&1; then
                            docker compose pull
                            docker compose up -d
                        elif command -v docker-compose >/dev/null 2>&1; then
                            docker-compose pull
                            docker-compose up -d
                        elif command -v docker >/dev/null 2>&1; then
                            ARCH=$(uname -m)
                            case "$ARCH" in
                                x86_64) COMPOSE_ARCH="docker-compose-linux-x86_64" ;;
                                aarch64|arm64) COMPOSE_ARCH="docker-compose-linux-aarch64" ;;
                                *) echo "Unsupported architecture for Compose bootstrap: $ARCH" >&2; exit 1 ;;
                            esac
                            PLUGIN_DIR="${HOME}/.docker/cli-plugins"
                            mkdir -p "$PLUGIN_DIR"
                            URL="https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/${COMPOSE_ARCH}"
                            if command -v curl >/dev/null 2>&1; then
                                curl -fsSL "$URL" -o "$PLUGIN_DIR/docker-compose"
                            elif command -v wget >/dev/null 2>&1; then
                                wget -qO "$PLUGIN_DIR/docker-compose" "$URL"
                            else
                                echo "Need curl or wget to install the Docker Compose v2 plugin." >&2
                                exit 1
                            fi
                            chmod +x "$PLUGIN_DIR/docker-compose"
                            docker compose pull
                            docker compose up -d
                        else
                            echo "docker is not installed on this agent." >&2
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
