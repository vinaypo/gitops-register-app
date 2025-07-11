pipeline {
  agent {label 'Jenkins-Agent'}
  tools {
    jdk 'Java17'
    maven 'Maven3'
  }
  environment {
    APP_NAME= "register-app-pipeline"
    DOCKER_USER= "vinay2806"
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG = "${BUILD_NUMBER}"
    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
        git branch: 'main', url: 'https://github.com/vinaypo/register-app'
      }
    }
    stage ("Build Application") {
      steps {
        sh 'mvn clean package'
      }
    }
    stage (" Test Application") { 
      steps {
        sh 'mvn test'
      }
    }
    stage ("SonarQube Analysis") {
      steps {
        script {
            withSonarQubeEnv(credentialsId: 'sonarqube') {
              sh 'mvn sonar:sonar'
          }
        }
      }
    }
    stage ("Quality Gate") {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
        }
      }
    }
    stage ("Build and Push Docker Image") {
      steps {
        script {
          withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
            def docker_image = docker.build("${IMAGE_NAME}")
            docker_image.push("${IMAGE_TAG}")
            docker_image.push("latest")
          }
        }
      }
    }
    stage ("Trivy Scan") {
      steps {
          sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table vinay2806/register-app-pipeline:latest')
      }
    }
    stage ("clean artifacts") {
      steps {
          sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
          sh "docker rmi ${IMAGE_NAME}:latest"
      }
    }
    stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-18-206-169-221.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops_token'"
                }
            }
       }
  }
}
