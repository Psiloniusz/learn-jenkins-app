pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                    reuseNode true
                }
            }
            steps {
                // cleanWs()
                // sh '''
                    sh 'ls -la'
                    sh 'npm config set cache "$(pwd)/.npm"'
                    sh 'node --version'
                    sh 'npm --version'
                    sh 'npm cache clean --force'
                    // sh 'npm ci'
                    // sh 'npm run build'
                    sh 'ls -la'
                // '''
            }
        }
    }
}
