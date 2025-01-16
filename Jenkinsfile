pipeline {
    agent any

    parameters {
        string(name: 'NETLIFY_SITE_ID', defaultValue: params.NETLIFY_SITE_ID ?: 'default-value', description: 'Netlify Site ID')
    }

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine3.20'
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

        stage('Run Parallel Tests') {
            failFast true 
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine3.20'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm test
                            # test -f "build/index.html"
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine3.20'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install -g @jsware/jsonpath-cli
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json --message "Deployed from Jenkins" --site $NETLIFY_SITE_ID | jpp '$.deploy_url'
                '''
            }
        }

        stage('Approval') {
            steps { 
                timeout(activity: true, time: 15) {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }   
        }

        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine3.20'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://mellifluous-phoenix-1d9e0c.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}