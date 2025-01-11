pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -alh
                    node --version
                    npm --version
                    npm install
                    npm ci
                    npm run build
                    ls -alh
                '''
            }
        }
    }
}
