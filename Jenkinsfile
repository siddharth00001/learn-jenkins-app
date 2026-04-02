pipeline {

    environment  {
        NETLIFY_SITE_ID = "c669eca6-00ec-4939-bde5-fc54dc3c79b2"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL = "https://comfy-madeleine-ea7926.netlify.app/"
    }

    agent any

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
                echo "Pipeline Started"
                ls -la 
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }
        stage('Tests') {
            parallel {

                stage('Unit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                            }
                        }
                        steps {
                            sh '''
                            echo "Test Stage"
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
                    
        stage('E2E') {
            agent{
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true

                        }
                }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 40
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent{
                    docker {
                        image 'node:18-alpine'
                        reuseNode true

                        }
                }
                    steps {
                        sh '''
                            npm install netlify-cli --ignore-scripts
                            NETLIFY_INSTANCE=./node_modules/.bin/netlify
                            $NETLIFY_INSTANCE --version
                            $NETLIFY_INSTANCE status
                            $NETLIFY_INSTANCE deploy \
                            --dir=build \
                            --prod \
                            --auth=$NETLIFY_AUTH_TOKEN \
                            --site=$NETLIFY_SITE_ID \
                            --no-build
                        '''
                    }
                }

        stage('Prod') {

                agent{
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true

                            }
                    }
                environment  {
                        CI_ENVIRONMENT_URL = "https://comfy-madeleine-ea7926.netlify.app"
                    }
                steps {
                        sh '''
                        npx playwright test --reporter=html
                        '''
                    }
                post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
        
    }
    
}