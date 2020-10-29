pipeline {
  agent any
  stages {
    stage('Run Unit Tests') {
      steps {
        powershell(script: """ 
          cd Server
          dotnet test
          cd ..
        """)
      }
    }
    stage('Docker Build') {
      steps {
        powershell(script: 'docker-compose build')     
        powershell(script: 'docker images -a')
      }
    }
    stage('Run Test Application') {
      steps {
        powershell(script: 'docker-compose up -d')    
      }
    }
    stage('Run Integration Tests') {
      steps {
        powershell(script: './Tests/ContainerTests.ps1') 
      }
    }
    stage('Stop Test Application') {
      steps {
        powershell(script: 'docker-compose down') 
        // powershell(script: 'docker volumes prune -f')   		
      }
      post {
	    success {
	      echo "Build successfull! You should deploy! :)"
	    }
	    failure {
	      echo "Build failed! You should receive an e-mail! :("
	    }
      }
    }
  }
}
