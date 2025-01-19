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
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                 }
         }

         stage('deploy'){
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
         
    }
    
}