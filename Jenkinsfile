def repoName = ''
def gitUrl = ''
def branchEnv = ''

pipeline {
    agent any

    environment {
        BRANCH_NAME = "${GIT_BRANCH}"
        GIT_URL = "${GIT_URL}"
        STACK_FILE = "stack.yaml"   // your stack file
        IMAGE_NAME = "ghcr.io/port-labs/port-self-hosted-github-app:latest"
    }

    stages {

        stage('Resolve Repository Info') {
            steps {
                script {
                    branchEnv = BRANCH_NAME
                    gitUrl = GIT_URL
                    repoName = gitUrl.tokenize('/').last().split("\\.")[0]
                }
                echo "Repo URL: ${gitUrl}"
                echo "Repo Name: ${repoName}"
                echo "Branch: ${branchEnv}"
            }
        }

        stage('Login to GHCR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-integration', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_TOKEN')]) {
                    sh """
                        echo $GHCR_TOKEN | docker login ghcr.io -u $GHCR_USER --password-stdin
                    """
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                echo "Pulling latest image ${IMAGE_NAME}"
                sh "docker pull ${IMAGE_NAME}"
            }
        }

        stage('Remove Docker Stack') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'qa'
                    branch 'master'
                }
            }
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
