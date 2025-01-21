pipeline {
    agent any

    environment {
            NETLIFY_SITE_ID ='91061868-5eec-497b-9383-f98d6e4b076d'
            NETLIFY_AUTH_TOKEN = credentials('netlify-token')
            REACT_APP_VERSION = '1.3.2'
    }

    stages {
       /* stage('build project') {

        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }

            steps {
                sh'''
                   echo 'Started building the project'
                   ls -la
                   node --version
                   npm --version
                   npm ci
                   npm run build
                   ls -la

                '''
            }
        } */
        stage ('Test'){
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

            post{
                always {
                         junit 'jest-results/junit.xml'
                       }
            }   
        }

        stage ('E2E'){

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                   sh '''
                       echo "added the polling SCM options"
                       npm install serve
                       node_modules/.bin/serve -s build &
                       sleep 10
                       npx playwright test  --reporter=html
                   '''
            }
            post {
                  always {
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                 }
         }

         stage ('Stage deploy E2E Test'){

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL ="STAGING URL TO BE SET"
            }

            steps {
                   sh '''
                     npm install netlify-cli node-jq
                     node_modules/.bin/netlify --version
                     echo "deploying into production with SITE ID :$NETLIFY_SITE_ID"
                     node_modules/.bin/netlify status
                     node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                     CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                     npx playwright test  --reporter=html
                '''
            }
            post {
                  always {
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Stage E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                 }
         }  

            stage ('Manual approval'){
                 steps {
                         timeout(time: 15, unit: 'MINUTES') {
                         input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                         }
                    }
            }
         
         stage ('PROD deploy E2E Test'){

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL ='https://nimble-genie-83fd80.netlify.app'
            }

            steps {
                   sh '''
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "deploying into production with SITE ID :$NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                        npx playwright test  --reporter=html
                   '''
            }
            post {
                  always {
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                 }
         }
         
    }
    
}