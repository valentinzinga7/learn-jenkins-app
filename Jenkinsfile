pipeline {
    agent any

    stages {
        stage('Test'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                test -f build/index.html
                npm test
                '''
            }
        }
        stage('E2E'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.46.1-jammy'
                    reuseNode true
                
                }
            }
            steps{
                sh '''
                npm install serve
                node_modules/.bin/serve -s build
                npx playwright test
                '''
            }
        }
    } 
}