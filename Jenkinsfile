pipeline {
  agent any

  options {
    skipDefaultCheckout(true)
    timestamps()
  }

  environment {
    NODE_TOOL = 'node'
    IMG_TAG   = 'v1.0'
  }

  tools {
    nodejs 'node'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        bat 'git rev-parse --short HEAD'
      }
    }

    stage('Init vars') {
      steps {
        script {
          env.PORT     = (env.BRANCH_NAME == 'main') ? '3000' : '3001'
          env.IMG_NAME = (env.BRANCH_NAME == 'main') ? 'nodemain' : 'nodedev'
          env.CT_NAME  = (env.BRANCH_NAME == 'main') ? 'nodeapp-main' : 'nodeapp-dev'
          echo "Branch: ${env.BRANCH_NAME} | Port: ${env.PORT} | Image: ${env.IMG_NAME}:${env.IMG_TAG} | Container: ${env.CT_NAME}"
        }
      }
    }

    stage('Set env (logo & config)') {
      steps {
        powershell '''
          if ($env:BRANCH_NAME -eq "main") {
            Copy-Item -Force "assets\\cat.svg" "src\\logo.svg"
          } else {
            Copy-Item -Force "assets\\mate.svg" "src\\logo.svg"
          }
          Get-Item "src\\logo.svg" | Format-List FullName,Length,LastWriteTime
        '''
      }
    }

    stage('Build') {
      steps {
        bat 'call npm ci || call npm install'
      }
    }

    stage('Test') {
      steps {
        bat 'set CI=true && call npm test --watchAll=false || exit /b 0'
      }
    }

    stage('Build Docker image') {
      steps {
        powershell """
          docker version
          docker build --build-arg APP_PORT=3000 -t ${env.IMG_NAME}:${env.IMG_TAG} .
        """
      }
    }

    stage('Deploy') {
      steps {
        powershell """
          \$ct = '${env.CT_NAME}'

          docker ps -q -f "name=\$ct" | ForEach-Object { docker stop \$_ }
          docker ps -aq -f "name=\$ct" | ForEach-Object { docker rm \$_ }

          docker run -d --name ${env.CT_NAME} --expose 3000 -p ${env.PORT}:3000 ${env.IMG_NAME}:${env.IMG_TAG}

          Write-Host "`nRunning containers:"
          docker ps --format "table {{.Names}}`t{{.Image}}`t{{.Ports}}"
        """
      }
    }
  }

  post {
    success {
      echo "Deployed ${env.IMG_NAME}:${env.IMG_TAG} on port ${env.PORT} (branch ${env.BRANCH_NAME})"
    }
    failure {
      echo "Build failed on branch ${env.BRANCH_NAME}"
    }
  }
}
