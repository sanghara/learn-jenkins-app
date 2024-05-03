pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'YOUR_NETLIFY_SITE_ID'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                script {
                    // Add 'script' block to handle shell commands
                    echo 'Building'
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
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent any
                    steps {
                        script {
                            echo 'Running unit tests'
                            sh 'npm test'
                        }
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
                        script {
                            echo 'Running end-to-end tests'
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent any
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            steps {
                script {
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    sh '''
                        npm install netlify-cli node-jq
                        node_modules/.bin/netlify --version
                        echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                        CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    '''
                }
            }
            post {
                always {
                    script {
                        echo "Staging environment deployed at: $CI_ENVIRONMENT_URL"
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
        }

        stage('Deploy prod') {
            agent any
            environment {
                CI_ENVIRONMENT_URL = 'YOUR_NETLIFY_SITE_URL'
            }
            steps {
                script {
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    sh '''
                        node --version
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                    '''
                }
            }
            post {
                always {
                    script {
                        echo "Production environment deployed at: $CI_ENVIRONMENT_URL"
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
        }
    }
}
