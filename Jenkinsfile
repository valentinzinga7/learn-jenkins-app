    pipeline {
            agent any

            environment {
                NETLIFY_SITE_ID = 'f1bca2df-1cd0-49a0-afb5-d9d9f4daa039'
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                        echo 'small changes'
                        ls -la
                        node --version
                        npm --version
                        npm ci
                        npm run build
                        ls -la
                    '''
                }
                
            }
            stage ('Tests'){
                parallel {
                    stage('Unit Tests'){
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
                post {
                always{
                    junit 'test-results/junit.xml'
                    }
                }
                }
                stage('E2E'){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        
                        }
                    }
                    steps{
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=line
                        '''
                    }
                    post {
                    always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
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
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Site ID: NETLIFY_SIDE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                    '''
                }
        }
        stage('Prod E2E'){
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                    
                    }
                environment {
                    CI_ENVIRONMENT_URL = 'https://cool-choux-a03687.netlify.app'
                            }
                }
                steps{
                    sh '''
                    npx playwright test --reporter=line
                    '''
                }
                post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
                }
            }
}
}