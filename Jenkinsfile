pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0efe79bf-128c-4425-9c55-a90f8c77bbb7'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
    }

    stages {
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Starting E2E tests'
                            npm install serve
                            node_modules/.bin/serve -s build &
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

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'small change testing SCM Polling'
                    echo 'Deploy Stage started'
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo 'Deploying to prod site with ID: $NETLIFY_SITE_ID'
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
            steps {
                sh '''
                    echo 'Starting Prod E2E tests'
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}
