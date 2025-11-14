def repoName = 'rancher'

pipeline {
    agent any

    environment {
        // Corrected to use the stack file name you provided
        STACK_FILE = "stack.yaml" 
        
        // Consistent with the image used in the stack.yaml
        IMAGE_NAME = "rancher/rancher:v2.10.1" 
        
        // SETUP_SCRIPT variable is now removed
    }

    stages {
        // The 'Setup Swarm Environment' stage has been removed

        stage('Pull Docker Image') {
            steps {
                echo "Pulling image ${IMAGE_NAME}"
                sh "docker pull ${IMAGE_NAME}"
            }
        }

        stage('Remove Docker Stack') {
            steps {
                echo "Removing Docker Stack ${repoName}"
                // Gracefully attempt to remove the stack if it exists
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh "docker stack rm ${repoName}"
                    sh "sleep 5" // Give the stack time to shut down
                }
            }
        }

        stage('Deploy Docker Stack') {
            steps {
                echo "Deploying Docker Stack ${repoName} using ${STACK_FILE}"
                // Deploying using the stack.yaml file
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
