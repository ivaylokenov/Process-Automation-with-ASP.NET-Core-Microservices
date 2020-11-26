pipeline {
  agent any
  stages {
    // stage('Verify Branch') {
    //   steps {
    //     echo "$GIT_BRANCH"
    //   }
    // }
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
    stage('Push Images') {
      when { branch 'main' }
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
            def image = docker.image("ivaylokenov/carrentalsystem-identity-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
        }
      }
    } 
    stage('Deploy Development') {
      when { branch 'main' }
      steps {
        powershell(script: 'kubectl config set-context gke_code-it-up-project_us-central1-c_car-rental-system-development')  
        powershell(script: 'kubectl apply -f .\.k8s\databases\')  
        powershell(script: 'kubectl apply -f .\.k8s\event-bus\')  
        powershell(script: 'kubectl apply -f .\.k8s\web-services\')  
      }
    }
  }
}
