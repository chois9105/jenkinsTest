pipeline {
    agent any 

    triggers {
        pollSCM('*/3 * * * *' )
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.'
    }

    stages {
        
        stage('Prepare'){
            agent any
            steps {
                echo "Lets start Long Journey! ENV: ${ENV}"
                git url: 'https://github.com/',
                    branch: 'master',
                    credentialsId: 'githubtest'
            }
        }

        post {
            success {
                echo 'Success'
            }

            always {
                echo 'I tried...'
            }

            cleanup {
                echo 'after all other post condition'
            }
        }

        stage('Deploy Frontend'){
            steps{

                echo 'Deploying frontend'...
                dir('./frontend'){
                    sh '''
                        aws s3 sync ./ s3://simjenkinsbucket
                    '''
                }
            }

            post {
                success {
                    echo 'Succes Deploying frontend'

                    mail to: 'csh778@nate.com',
                        subject: 'Deploy frontend success',
                        body: 'success'
                }
            }

        }

        stage('start backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./cra'){
                    sh '''
                    npm install &&
                    npm start
                    '''
                }
            }

        }

        stage('build backend'){
            agent any
            steps {

                echo 'build backend'
                dir('./cra'){
                    sh """
                    docker build . -t cra --build-arg env=${PROD}
                    """
                }

            }

            post {
                failure {
                    error 'Bomb'
                }
            }

        }

        stage('Deploy backend'){
            agent any

            echo 'build backend'
            dir('./cra'){
                sh """
                docker run -p 80:80 -d cra
                """
            }
        }




    }

}