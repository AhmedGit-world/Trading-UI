pipeline {
    agent any
      

    stages {
        stage('Git checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/AhmedGit-world/Trading-UI.git'
                   }
}
        stage('Install npm prerequisites'){
            steps{
                sh'npm audit fix'
                sh'npm install'
                sh'npm run build'
                sh'cd /var/lib/jenkins/workspace/Trading-ui-pipeline/build'
                sh'pm2 --name Trading-UI start npm -- start'
            }
        }
    }
}
