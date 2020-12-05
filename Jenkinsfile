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
        powershell(script: 'docker-compose build')
        powershell(script: 'docker build -t pesho1/carrentalsystem-user-client-development --build-arg configuration=development .\\Client\\')
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
      when { anyOf { branch 'main'; branch 'development' } }
      steps {
        script {
          def images = [
            "pesho1/carrentalsystem-identity-service",
            "pesho1/carrentalsystem-dealers-service",
            "pesho1/carrentalsystem-statistics-service",
            "pesho1/carrentalsystem-notifications-service",
            "pesho1/carrentalsystem-user-client",
            "pesho1/carrentalsystem-admin-client",
            "pesho1/carrentalsystem-watchdog-service",
            "pesho1/carrentalsystem-user-client-development"
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
    stage('Deploy Development') {
      //when { branch 'development' }
      when { branch 'main' }
      steps {
        withKubeConfig([credentialsId: 'DevelopmentServer', serverUrl: 'car-rental-system-dns-18b73077.hcp.westeurope.azmk8s.io']) {
          powershell(script: 'kubectl apply -f ./.k8s/.environment/development.yml')
          powershell(script: 'kubectl apply -R -f ./.k8s/objects/')
        }
      }
    }
  }
  // post {
  //   always {
  //       emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
  //         recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
  //         subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
  //   }
  // }
}
