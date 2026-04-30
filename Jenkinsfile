pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID ='f0eeed19-c602-4eb2-9a3c-36fa0850574f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        /*stage('Install & Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm config set fetch-timeout 120000
                    npm config set fetch-retry-mintimeout 20000
                    npm config set fetch-retry-maxtimeout 120000
                    npm ci
                    npm run build
                '''
            }
        }*/

        stage('Tests in Parallel') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test -- --coverage
                        '''
                    }
                    post {
                        always {
                            junit allowEmptyResults: true, testResults: '**/junit.xml'
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'coverage',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
                        }
                    }
                }

                stage('E2E Tests') {
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
                            SERVE_PID=$!
                            sleep 10
                            npx playwright test --reporter=html,list || true
                            kill $SERVE_PID 2>/dev/null || true
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Report'
                            ])
                        }
                    }
                }
            }
        }

        stage('Netlify Deploy on Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm config set fetch-timeout 120000
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        stage('Manual Approval'){
            steps{
                timeout(5) {
                        input 'Are you ready for Deploying?'
                }
            }
        }
        stage('Netlify Deploy on Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm config set fetch-timeout 120000
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir=build
                '''
            }
        }

        stage('E2E Prod Tests') {
            environment {
                CI_ENVIRONMENT_URL='https://heartfelt-malasada-564a26.netlify.app'
            }
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npx playwright test --reporter=html,list 
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report-prod',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Report Prod'
                    ])
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/junit.xml'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}