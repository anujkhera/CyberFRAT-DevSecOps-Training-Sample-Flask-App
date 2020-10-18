pipeline {
  environment {
    registry = "anujkhera/cyberfrat"
    registryCredential = "DockerHub"
    dockerImage = ''
  }
  agent any
  
  stages {
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
          }
      }
    }
    stage('Push to DockerHub') {
      steps {
        script{
          docker.withRegistry('', registryCredential ) {
          docker.Image.push()
        }
      }
     }
    }   
        
    stage('Test Run') {
      steps {
        sh 'docker run -d cyberfrat:$BUILD_NUMBER'
      }
    }
   }
 }
