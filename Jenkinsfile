pipeline {
  environment {
    registry = "anujkhera/cyberfrat"
    registryCredential = "DockerHub"
    dockerImage = ''
  }
  agent any
  
   stages {
    stage('Check for Secrets'){
      steps {
        sh "rm -rf trufflehog.json || true"
        sh "docker run dxa4481/trufflehog:latest --json https://github.com/anujkhera/CyberFRAT-DevSecOps-Training-Sample-Flask-App.git > trufflehog.json || true"
        sh "cat trufflehog.json"
      }
    }
  
      
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
          dockerImage.push()
        }
      }
     }
    }   
        
    stage('Test Run') {
      steps {
        sh 'docker run -d $registry:$BUILD_NUMBER'
      }
    }
   }
 }
