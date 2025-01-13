pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
