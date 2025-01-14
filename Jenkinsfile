pipeline {
    agent any

    stages {
        stage('build project') {

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
        }
        stage ('test'){
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
        }
    }
}