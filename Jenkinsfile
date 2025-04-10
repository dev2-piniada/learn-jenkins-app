pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '7257c559-8b50-4ba1-84f7-914bfdf211c8'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_TOKEN')
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

        stage ('Run Tests') {
            parallel {
                stage('Unit Tests') {
                   agent {
                       docker {
                           image 'node:18-alpine'
                           reuseNode true
                       }
                   }

                   steps {
                       sh '''
                           #test -f build/index.html
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
                   agent {
                       docker {
                           image 'my-playwright'
                           reuseNode true
                       }
                   }

                   steps {
                       sh '''
                           serve -s build &
                           sleep 10
                           npx playwright test --reporter=html
                       '''
                   }

                    post {
                       always {
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                       }
                    }
                }
            }
       }



        stage('Deploy Staging & E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r \'.deploy_url\' deploy-output.json)
                    echo "Deployed to staging. URL: $CI_ENVIRONMENT_URL"
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        /*
        commented manual approval for continuous deployment

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        } */

        stage('Deploy Production & E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://harmonious-palmier-a0526b.netlify.app/'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
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
