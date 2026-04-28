pipeline {
    agent any

    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    whoami
                    pwd
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                    pwd
                '''
            }
        }
        */

        stage('Test') {
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
                    npm install -g serve
                    serve -s build
                    npx playwrite test
                '''
            }
        }

    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
