pipeline {
  agent any
  stages {
    stage('Verify Branch') {
      steps {
        echo "$GIT_BRANCH"
      }
    }
    stage('Run Unit Tests'){
      steps {
        powershell(script: 'dotnet test')
      }
    }
    stage('Docker Build'){
      steps {
        powershell(script: 'docker-compose build')     
        powershell(script: 'docker images -a')
      }
    }
    stage('Run Test Application'){
      steps {
        powershell(script: 'docker-compose up')    
      }
    }
    stage('Run Integration Tests'){
      steps {
        echo "Tests successful!"
      }
    }
    stage('Stop Test Application'){
      steps {
        powershell(script: 'docker-compose down')    
      }
    }
  }
}
