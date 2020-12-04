pipeline {
  agent any
  stages {
    stage('Verify Branch') {
      steps {
        echo "${GIT_BRANCH}"
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
        powershell(script: 'docker-compose build -e configuration=prod')
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
          def images = [
            "pesho1/carrentalsystem-identity-service",
            "pesho1/carrentalsystem-dealers-service",
            "pesho1/carrentalsystem-statistics-service",
            "pesho1/carrentalsystem-notifications-service",
            "pesho1/carrentalsystem-user-client",
            "pesho1/carrentalsystem-admin-client",
            "pesho1/carrentalsystem-watchdog-service"
          ]

          docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
            for(imageName in images) {
              echo "Pushing ${imageName}"
              def image = docker.image(imageName)
              image.push("1.0.${env.BUILD_ID}")
              image.push('latest')
            }
          }
        }
      }
    }
  }
}
