def repoName = 'getport-test'

pipeline {
    agent any

    environment {
        STACK_FILE = "stack.yaml"
        IMAGE_NAME = "ghcr.io/port-labs/port-self-hosted-github-app:0.16.7"
    }

    stages {

        stage('Login to GHCR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-integration', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_TOKEN')]) {
                    sh """
                        echo \$GHCR_TOKEN | docker login ghcr.io -u \$GHCR_USER --password-stdin
                    """
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                echo "Pulling image ${IMAGE_NAME}"
                sh "docker pull ${IMAGE_NAME}"
            }
        }

        stage('Remove Docker Stack') {
            steps {
                echo "Removing Docker Stack ${repoName}"
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh "docker stack rm ${repoName}"
                }
            }
        }

        stage('Deploy Docker Stack') {
            steps {
                echo "Deploying Docker Stack ${repoName}"
                sh "docker stack deploy -c ${STACK_FILE} ${repoName}"
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
