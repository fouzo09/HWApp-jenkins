pipeline{
    environment{
        IMAGE_NAME="alpinehelloworld"
        IMAGE_TAG="latest"
        STAGING="fouzo_staging"
        PRODUCTION="fouzo_production"
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
            agent any
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
