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
        sh "docker rundxa4481/trufflehog:latest --json https://github.com/anujkhera/CyberFRAT-DevSecOps-Training-Sample-Flask-App.git > trufflehog.json || true"
        sh "cat trufflehog.json"
      }
    }
    
    stage('SCA'){
      steps {
        sh "pip3 install safety"
        sh "rm -rf safety.json || true"
        sh "safety check -r requirements.txt --json > safety.json || true"
        sh "cat safety.json"
      }
    }
    
    stage('SAST'){
      steps {
        sh "rm -rf bandit.json || true"
        sh "bandit -r -f=json -o=bandit.json . || true"
        sh "cat bandit.json"
      }
    }
    
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    
    stage('Push to DockerHub') {
      steps {
        script {
          docker.withRegistry('', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    
    stage('Deploy Test Application') {
      steps {
        sh 'docker stop flaskr && docker rm flaskr || true'
        sh 'docker pull anujkhera/cyberfrat:$BUILD_NUMBER'
        sh 'docker run -d -p 5000:5000 --name flaskr anujkhera/cyberfrat:$BUILD_NUMBER'
      }
    }

    stage('DAST Scan'){
      steps{
        sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://build.dsy.sh:5000/ || true'
      }
    }
    
    stage("Docker Security"){
      parallel{
        stage("Docker Benchmarking"){
          steps{
            sh '''
            docker pull docker/docker-bench-security
            docker run --net host --pid host --cap-add audit_control -v /var/lib:/var/lib -v /var/run/docker.sock:/var/run/docker.sock -v /usr/lib/systemd:/usr/lib/systemd -v /etc:/etc --label docker_bench_security docker/docker-bench-security || true
            '''
          }
        }
        
        stage("Image Scanning"){
          steps{
            sh '''
            docker run -d --name db arminc/clair-db || true
            sleep 15 # wait for db to come up
            docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
            sleep 1
            DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
            wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
            ./clair-scanner --ip="$DOCKER_GATEWAY" anujkhera/cyberfrat:$BUILD_NUMBER || exit 0
            '''
          }
        }
      }
    } 
  } 
}
