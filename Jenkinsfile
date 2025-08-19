pipeline {
  agent any

  options {
    skipDefaultCheckout(true)
    timestamps()
  }

  environment {
    NODEJS = 'node'
    APP_NAME = 'nodeapp'
    BRANCH = "${env.BRANCH_NAME}"
    PORT      = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    IMG_NAME  = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
    IMG_TAG   = "v1.0"
    CT_NAME   = "${env.BRANCH_NAME == 'main' ? 'nodeapp-main' : 'nodeapp-dev'}"
  }

  tools {
    nodejs "${env.NODEJS}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'git rev-parse --short HEAD'
      }
    }

    stage('Set env (logo & config)') {
      steps {
        script {
          if (BRANCH == 'main') {
            sh 'cp -f assets/cat.svg src/logo.svg'
          } else {
            sh 'cp -f assets/mate.svg src/logo.svg'
          }
        }
      }
    }

    stage('Build') {
      steps {
        sh 'npm ci || npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test --silent || true' 
      }
    }

    stage('Build Docker image') {
      steps {
        sh """
          docker build \
            --build-arg APP_PORT=3000 \
            -t ${IMG_NAME}:${IMG_TAG} .
        """
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh """
            docker ps -q --filter "name=${CT_NAME}" | xargs -r docker stop
            docker ps -aq --filter "name=${CT_NAME}" | xargs -r docker rm

            docker run -d --name ${CT_NAME} \
              --expose 3000 -p ${PORT}:3000 \
              ${IMG_NAME}:${IMG_TAG}
          """
        }
      }
    }
  }

  post {
    success {
      echo "Deployed ${IMG_NAME}:${IMG_TAG} on port ${PORT}"
    }
    failure {
      echo "Build failed on branch ${BRANCH}"
    }
  }
}
