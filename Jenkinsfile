pipeline{
    environment{
        IMAGE_NAME="alpinehelloworld"
        IMAGE_TAG="latest"
        STAGING="training-fouzohb-staging"
        PRODUCTION="training-fouzohb-production"
    }
    agent none
    stages{
        stage("Build image"){
            agent any
            steps{
                script{
                    sh  '''
                            docker run -d -p 5001:5000 -e PORT=5000 --name $IMAGE_NAME  eazytraining/$IMAGE_NAME:$IMAGE_TAG
                            sleep 5
                        '''
                }
            }
        }

        stage("Test image"){
            agent any
            steps{
                script{
                    sh  '''
                            curl http://172.28.128.6:5001 | grep -q "Hello world!"
                        '''
                }
            }
        }

        stage("Remove container"){
            agent any
            steps{
                script{
                    sh  '''
                            docker stop $IMAGE_NAME
                            docker rm $IMAGE_NAME
                        '''
                }
            }
        }

        stage("Pusher image to staging and deploy it"){
            when {
                expression { GIT_BRANCH == "origin/master" }
            }
            agent any
            environment{
                HEROKU_API_KEY=credentials("heroku_api_key")
            }
            steps{
                script{
                    sh  '''
                            heroku container:login
                            heroku create $STAGING
                            heroku container:push -a $STAGING web
                            heroku container:release -a $STAGING web
                        '''
                }
            }
        }

        stage("Pusher image to production and deploy it"){
            when {
                expression { GIT_BRANCH == "origin/master" }
            }
            agent any
            environment{
                HEROKU_API_KEY=credentials("heroku_api_key")
            }
            steps{
                script{
                    sh  '''
                            heroku container:login
                            heroku create $PRODUCTION
                            heroku container:push -a $PRODUCTION web
                            heroku container:release -a $PRODUCTION web
                        '''
                }
            }
        }
    }
    post {
       success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
         }
      failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }   
    }
}
