def repoName = ''
def gitUrl = ''
def branchEnv = ''

pipeline {
    agent any

    environment {
        BRANCH_NAME = "${GIT_BRANCH}"
        GIT_URL = "${GIT_URL}"
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
                withCredentials([usernamePassword(credentialsId: 'GHCR_CREDS', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_TOKEN')]) {
                    sh """
                        echo $GHCR_TOKEN | docker login ghcr.io -u $GHCR_USER --password-stdin
                    """
                }
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
                sh "docker stack deploy -c stack.yaml ${repoName}"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
