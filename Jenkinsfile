pipeline {
  agent any
  stages {
    stage('Docker Build') {
      steps {
        powershell(script: 'docker-compose build')     
        powershell(script: 'docker images -a')
      }
    }
  }
}
