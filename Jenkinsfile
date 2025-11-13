def repoName = 'rancher'

pipeline {
    agent any

    environment {
        STACK_FILE = "stack.yaml"
        IMAGE_NAME = "rancher/rancher:latest"
    }

    stages {

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
