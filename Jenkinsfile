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
        
        stage('Tests'){
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Project ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build 
                '''
                script {
                env.STAGING_URL = sh(script: 'node_modules/.bin/node-jq -r ".deploy_url" deploy-output.json', returnStdout:true)
                }
            }
        }
        stage('Staging E2E Test') {
            agent {
                docker {
                        image 'mcr.microsoft.com/playwright:v1.53.1-jammy'
                        reuseNode true
                        }
                    }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Manual Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }
        stage('Deploy Prod') {
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
        stage('Prod E2E Test') {
            agent {
                docker {
                        image 'mcr.microsoft.com/playwright:v1.53.1-jammy'
                        reuseNode true
                        }
                    }
            environment {
                CI_ENVIRONMENT_URL = 'https://wondrous-heliotrope-585c4a.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }           
}
