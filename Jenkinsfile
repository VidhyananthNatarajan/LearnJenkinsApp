pipeline {
    agent any

    environment {
            NETLIFY_SITE_ID ='91061868-5eec-497b-9383-f98d6e4b076d'
            NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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

         stage('deploy to stage'){
            agent {
               docker {
                image 'node:18-alpine'
                reuseNode true
                }
            }

            steps{
                 sh '''
                     npm install netlify-cli node-jq
                     node_modules/.bin/netlify --version
                     echo "deploying into production with SITE ID :$NETLIFY_SITE_ID"
                     node_modules/.bin/netlify status
                     node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.Stage_URL = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout:true)
                }
            }
         }  
         stage ('Stage E2E Test'){

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL ="${env.Stage_URL}"
            }

            steps {
                   sh '''
                       echo $CI_ENVIRONMENT_URL
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
         

         stage('deploy to prod'){
            agent {
               docker {
                image 'node:18-alpine'
                reuseNode true
                }
            }

            steps{
                 sh '''
                     npm install netlify-cli
                     node_modules/.bin/netlify --version
                     echo "deploying into production with SITE ID :$NETLIFY_SITE_ID"
                     node_modules/.bin/netlify status
                     node_modules/.bin/netlify deploy --dir=build --prod
                 '''
            }


         }
         stage ('PROD E2E Test'){

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