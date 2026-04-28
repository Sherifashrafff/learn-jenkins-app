pipeline {
    agent any

    stages {
       /* stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }*/

        stage('Tests In Paralle'){
            Paralle{
                    stage('Test') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps {
                            sh '''
                                test -f build/index.html
                                npm test
                            '''
                        }
                    }

                    stage('E2E') {
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
                                npx playwright test --reporter=html
                                kill $SERVE_PID 2>/dev/null || true
                            '''
                        }
                    }
            }
        }

    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}