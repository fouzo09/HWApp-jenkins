pipeline{
    environment{
        IMAGE_NAME="alpinehelloworld"
        IMAGE_TAG="latest"
        STAGING="fouzo09_staging"
        PRODUCTION="fouzo09_production"
    }
    agent none
    stages{
        stage("Build image"){
            agent none
            steps{
                script{
                    sh  '''
                            docker run --name $IMAGE_NAME -d-p 5001:5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
                            sleep 5
                        '''
                }
            }
        }

        stage("Test image"){
            agent none
            steps{
                script{
                    sh  '''
                            curl http://172.28.128.6:5001 | grep -q "Hello world!"
                        '''
                }
            }
        }

        stage("Remove container"){
            agent none
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
            agent none
            environment{
                HEROKU_API_KEY=crendentials("heroku_api_key")
            }
            steps{
                script{
                    sh  '''
                            heroku container:login
                            heroku create $STAGING || echo "Projet disponible"
                            heroku container push -a $STAGING web
                            heroku container:release -a $STAGING web
                        '''
                }
            }
        }

        stage("Pusher image to production and deploy it"){
            when {
                expression { GIT_BRANCH == "origin/master" }
            }
            agent none
            environment{
                HEROKU_API_KEY=crendentials("heroku_api_key")
            }
            steps{
                script{
                    sh  '''
                            heroku container:login
                            heroku create $PRODUCTION || echo "Projet disponible"
                            heroku container push -a $PRODUCTION web
                            heroku container:release -a $PRODUCTION web
                        '''
                }
            }
        }
    }
}
