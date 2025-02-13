pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/SabbirSaleh/HFJen.git'
        REPO_BRANCH = 'main'
        FABRIC_CFG_PATH = "${WORKSPACE}/fabric-samples/test-network/config"
        PATH = "${WORKSPACE}/bin:${env.PATH}" // Ensure Fabric binaries are in PATH
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo "🔄 Cloning Repository from ${REPO_URL}"
                    git branch: "${REPO_BRANCH}", url: "${REPO_URL}"
                }
            }
        }

        stage('Install Fabric Binaries') {
    steps {
        script {
            echo "Setting up Hyperledger Fabric Binaries"
            sh '''
            export PATH=$WORKSPACE/bin:$PATH
            echo "Updated PATH: $PATH"
            which configtxgen  # Debug to check if binaries are accessible
            which peer  # Debug to check peer command availability
            '''
        }
    }
}


        stage('Set Up Fabric Network') {
    steps {
        script {
            echo "Bringing Down Any Existing Network"
            sh 'chmod +x network.sh'
            sh './network.sh down || echo "Network already down"'

            echo "Starting Fabric Network"
            sh 'chmod +x network.sh'
            sh 'ls -l'  # Debugging step to see if network.sh exists
            sh './network.sh up createChannel'
        }
    }
}


        stage('Deploy Chaincode') {
            steps {
                script {
                    echo "📦 Deploying Chaincode"
                    sh './network.sh deployCC -ccn basic -ccp ./asset-transfer-basic/chaincode-go -ccl go'

                    echo "✅ Chaincode Deployed"
                }
            }
        }

        stage('Verify Network & Chaincode') {
            steps {
                script {
                    echo "🔍 Checking Running Docker Containers"
                    sh 'docker ps'

                    echo "🔍 Querying Installed Chaincode"
                    sh 'docker exec cli peer lifecycle chaincode queryinstalled'

                    echo "🔍 Querying Committed Chaincode"
                    sh 'docker exec cli peer lifecycle chaincode querycommitted --channelID mychannel --name basic'
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    echo "🧹 Stopping the Network"
                    sh './network.sh down || echo "Network already down"'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline execution completed successfully!'
        }
        failure {
            echo '❌ Pipeline execution failed!'
        }
    }
}
