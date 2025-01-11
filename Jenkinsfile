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
                cleanWs()
                sh '''
                    npm config set cache "$(pwd)/.npm"
                    ls -la
                    node --version
                    npm --version
                    npm cache clean --force
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
