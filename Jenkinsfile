pipeline {
  agent any
  stages {
    stage('Verify Branch') {
      steps {
        echo "$GIT_BRANCH"
      }
    }
    stage('Docker Build'){
      steps {
        powershell(script: 'docker-compose build')     
        powershell(script: 'docker images -a')
      }
    }
  }
}
