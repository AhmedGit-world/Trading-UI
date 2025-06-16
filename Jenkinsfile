// Define a declarative pipeline for your Node.js application
pipeline {
    agent any

    environment {
        NODE_VERSION_TOOL_NAME = 'Node.js-24.2.0' // MUST match your Jenkins tool name
        APP_NAME = 'Trading-UI'
    }

    stages {
        // Stage 1: Checkout Source Code
        // Jenkins automatically checks out the repository to read the Jenkinsfile.
        // The workspace will already contain your code.
        // No explicit 'git' step is needed here.
        stage('Checkout Source Code') {
            steps {
                echo "Repository already checked out by Jenkins itself. Proceeding..."
            }
        }

        // Stage 2: Install Node.js Dependencies and Build
        stage('Install & Build') {
            steps {
                script {
                    withEnv(["PATH+NODE=${tool NODE_VERSION_TOOL_NAME}/bin"]) {
                        echo "Running npm audit fix..."
                        sh 'npm audit fix'

                        echo "Running npm install..."
                        sh 'npm install --force'

                        echo "Running npm run build..."
                        def packageJson = readJSON text: readFile(file: 'package.json')
                        if (packageJson.scripts && packageJson.scripts.build) {
                            sh 'npm run build'
                        } else {
                            echo "No 'build' script found in package.json. Skipping build step."
                        }
                    }
                }
            }
        }

        // Stage 3: Run Tests
        stage('Run Tests') {
            steps {
                script {
                    withEnv(["PATH+NODE=${tool NODE_VERSION_TOOL_NAME}/bin"]) {
                        echo "Running npm test..."
                        sh 'npm test'
                    }
                }
            }
        }

        // Stage 4: Deploy Application
        stage('Deploy Application') {
            when {
                // *** IMPORTANT CHANGE HERE: Targeting 'master' branch ***
                branch 'master'
            }
            steps {
                script {
                    echo "Starting deployment of ${env.APP_NAME} for branch ${env.BRANCH_NAME}..."

                    withEnv(["PATH+NODE=${tool NODE_VERSION_TOOL_NAME}/bin"]) {
                        // Ensure PM2 is installed globally on your Jenkins agent (npm install -g pm2)
                        sh "pm2 delete ${APP_NAME} || true"
                        sh "pm2 start npm --name ${APP_NAME} -- start"
                        sh "pm2 save"
                    }

                    echo "Deployment of ${APP_NAME} completed via PM2."
                }
            }
        }
    }

    post {
        always {
            deleteDir()
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
