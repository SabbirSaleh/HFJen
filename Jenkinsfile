pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/SabbirSaleh/HFJen.git'
        REPO_BRANCH = 'main'
        FABRIC_CFG_PATH = "${WORKSPACE}/fabric-samples/test-network/config"
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning Repository from ${REPO_URL}"
                git branch: "${REPO_BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Set Up Fabric Network') {
            steps {
                script {
                    echo "Bringing Down Any Existing Network"
                    sh './network.sh down || true'

                    echo "Starting Fabric Network"
                    sh 'chmod +x network.sh'
                    sh './network.sh up createChannel'
                }
            }
        }

        stage('Deploy Chaincode') {
            steps {
                script {
                    echo "Deploying Chaincode"
                    sh './network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go'
                }
            }
        }

        stage('Verify Network & Chaincode') {
            steps {
                script {
                    echo "Checking Running Docker Containers"
                    sh 'docker ps'

                    echo "Querying Installed Chaincode"
                    sh 'peer lifecycle chaincode queryinstalled'

                    echo "Query Committed Chaincode"
                    sh 'peer lifecycle chaincode querycommitted --channelID mychannel --name basic'
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    echo "Stopping the Network"
                    sh './network.sh down'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline execution completed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
    }
}

