pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0efe79bf-128c-4425-9c55-a90f8c77bbb7'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test Stage') {
            parallel {
                stage('UnitTest') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Test Stage started'
                            #test -f build/index.html
                            #ls -la build/index.html
                            npm test -- --watchAll=false
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-custom-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Starting E2E tests'
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-custom-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "Test Value for Staging URL to fix issue with localhost connection refused by Playwright tests"
            }

            steps {
                sh '''
                    echo 'Deploy to Staging environment started'
                    netlify --version
                    echo 'Deploying to staging site with ID: $NETLIFY_SITE_ID'
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
        //         }
        //     }
        // }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-custom-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://magnificent-klepon-2226b6.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    echo 'Deploying to prod site with ID: $NETLIFY_SITE_ID'
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}
