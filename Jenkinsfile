pipeline {
    agent {label 'Jenkins-Agent'}
    environment {
        APP_NAME = "register-app-pipeline"
    }
    stages {
        stage ("Clean Workspace") {
          steps {
             cleanWs()
        }
       }
       stage ("Checkout SCM"){
        steps {
          sh 'echo passed'
          git branch: 'main', url: 'https://github.com/vinaypo/gitops-register-app'
        }   
       }
        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }
        stage("Push the changes to SCM") {
            environment {
                GIT_REPOSITORY = "gitops-register-app"
                GIT_USERNAME = "vinaypo"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'github')]) {
                sh ''' 

                   git config  user.name "vinaypo"
                   git config  user.email "vapodila@gmail.com"
                   git add deployment.yaml
                   git commit -m "Update deployment manifest with image tag ${IMAGE_TAG}"
                   git push https://${github}@github.com/${GIT_USERNAME}/${GIT_REPOSITORY} HEAD:main
                '''

                }
              }
            }
    }
}

