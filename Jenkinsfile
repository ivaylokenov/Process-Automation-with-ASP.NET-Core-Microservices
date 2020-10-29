pipeline {
  agent any
  stages {
    stage('Verify Branch') {
      steps {
        echo "$GIT_BRANCH"
      }
    }
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
      }
      post {
	success {
	  echo "Build successfull! :)"
	}
	failure {
	  echo "Build failed! :("
	}
      }
    }
	stage('Push Container') {
	  steps {
		dir("$WORKSPACE/Server/CarRentalSystem.Identity") {
		  script {
			docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
			  def image = docker.build("ivaylokenov/carrentalsystem-identity-service:${env.BUILD_ID}")
			  
			  image.push()

			  image.push('latest')
			}
		  }
		}
	  }
	}
  }
}
