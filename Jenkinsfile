pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'b7ffc4ae-f146-4014-bf6c-adc8da541a4f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') //exact id from jenkins credentials
    }
    //here the stages start
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
                    echo "Small change to trigger Jenkins job"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Run Tests'){
            parallel{
                stage('Unit Test') {
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
                         post {
                            always {
                                junit 'jest-results/junit.xml'
                            }
                        }
                }
                stage('E2E Test') {
                    agent {
                            docker {
                                    image 'mcr.microsoft.com/playwright:v1.53.1-jammy'
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
                        npm install netlify-cli@20.1.1
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Project ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod 
                    '''
            }
        }

    }           
}
