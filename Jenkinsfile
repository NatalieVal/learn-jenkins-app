pipeline {
    agent any
    environment {
        BUILD_FILE_NAME = 'index.html'
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
        stage('Test') {
            steps {
                
                sh '''
                echo "Testing the build"
                test -f build/$BUILD_FILE_NAME
                npm test
                '''
            }
        }

    }
}