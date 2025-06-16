// Define a declarative pipeline for your Node.js application
pipeline {
    // Specifies where the pipeline will run.
    // 'agent any' means Jenkins can use any available agent.
    // For a Node.js specific environment, you might use 'agent { label 'nodejs-agent' }'
    // if you have a specific agent configured with Node.js.
    agent any

    // Define environment variables that can be used throughout the pipeline.
    environment {
        // Specify the Node.js version to use.
        // This MUST match the name of the Node.js installation configured in Jenkins
        // (Manage Jenkins -> Tools -> NodeJS installations...).
        // For example, if you named it 'Node.js-24.2.0', then use that exact string.
        NODE_VERSION_TOOL_NAME = 'Node.js-24.2.0' // Make sure this matches your Jenkins tool name
        APP_NAME = 'Trading-UI' // Define your application name for PM2
    }

    // Define the stages of your CI/CD pipeline
    stages {
        // Stage 1: Checkout the source code from the Git repository
        stage('Checkout Source Code') {
            steps {
                // Uses the Git SCM to clone the repository.
                // The URL and credentials will be configured in the Jenkins job itself.
                git branch: 'main', url: 'https://github.com/AhmedGit-world/Trading-UI.git'
            }
        }

        // Stage 2: Install Node.js Dependencies and Build
        stage('Install & Build') {
            steps {
                script {
                    // Ensure Node.js and npm are in the PATH using the configured Jenkins tool
                    withEnv(["PATH+NODE=${tool NODE_VERSION_TOOL_NAME}/bin"]) {
                        echo "Running npm audit fix..."
                        sh 'npm audit fix' // Running audit fix first

                        echo "Running npm install..."
                        sh 'npm install' // Install all dependencies

                        echo "Running npm run build..."
                        def packageJson = readJSON text: readFile(file: 'package.json')
                        if (packageJson.scripts && packageJson.scripts.build) {
                            sh 'npm run build' // Execute build script if it exists
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
                        sh 'npm test' // Run tests
                    }
                }
            }
        }

        // Stage 4: Deploy Application
        stage('Deploy Application') {
            // This 'when' condition ensures the deploy stage only runs for builds on the 'main' branch.
            when {
                branch 'main'
            }
            steps {
                script {
                    echo "Starting deployment of ${env.APP_NAME} for branch ${env.BRANCH_NAME}..."

                    // Ensure Node.js and npm are in the PATH for PM2 commands
                    withEnv(["PATH+NODE=${tool NODE_VERSION_TOOL_NAME}/bin"]) {
                        // Navigate to the build directory if your 'npm start' requires it to be run
                        // from a specific 'build' or 'dist' folder.
                        // You might need to adjust this path based on where 'npm run build' outputs files.
                        // If 'npm start' works from the root, you can remove this 'cd' command.
                        // sh "cd ${WORKSPACE}/build" // Example if build output is in 'build' subdirectory

                        // Stop/delete existing PM2 process and start a new one
                        // Ensure PM2 is installed globally on your Jenkins agent or
                        // accessible via PATH (e.g., if installed via npm -g)
                        sh "pm2 delete ${APP_NAME} || true" // Stop/delete previous instance, '|| true' prevents failure if not running
                        sh "pm2 start npm --name ${APP_NAME} -- start" // Start the app using npm start
                        sh "pm2 save" // Save PM2 process list to be persistent across reboots
                    }

                    echo "Deployment of ${env.APP_NAME} completed via PM2."
                }
            }
        }
    }

    // Post-build actions: run cleanup or notifications after the pipeline finishes
    post {
        always {
            // Clean up workspace after build, highly recommended to keep your Jenkins server clean
            deleteDir()
        }
        success {
            echo 'Pipeline finished successfully!'
            // Add notification (e.g., Slack, email) here if desired
        }
        failure {
            echo 'Pipeline failed!'
            // Add failure notification here
        }
    }
}
