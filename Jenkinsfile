pipeline {
      environnement  {
          IMAGE_NAME = "alpinehelloworld"
          IMAGE_TAG = "latest"
          STAGING = "eazytraining-staging"
          PRODUCTION = "eazytraining-production"
        }
        agent none
        stages {
            stage('Build image') {
                agent any
                steps {
                   script {
                      sh 'docker build -t nyona77/$IMAGE_NAME:$IMAGE_TAG .'
                      
                   }
                }
            }
            stage('Run Container base on builded image') {
                agent any
                steps {
                   script {
                      sh '''
                          docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 nyona77/$IMAGE_NAME:$IMAGE_TAG
                          sleep 5
                      '''
                      
                   }
                }
            }
            stage('Test image') {
                agent any
                steps {
                   script {
                      sh '''
                          curl http://localhost | grep -q "Hello world!"
                      '''
                      
                   }
                }
            }
            stage('Clean Container') {
                agent any
                steps {
                   script {
                      sh '''
                          docker stop $IMAGE_NAME
                          docker rm $IMAGE_NAME
                      '''
                      
                   }
                }
            }
            stage('Push image in production and deploy it') {
              when {
                      expression { GIT_BRANCH == 'origin/master' }
                   }
              agent any
              environment {
                    HEROKU_API_KEY = credentials('heroku_api_key')
              }
              steps {
                  script {
                      sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container: push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                      '''
                  }
              }
              
            }
        }
}
