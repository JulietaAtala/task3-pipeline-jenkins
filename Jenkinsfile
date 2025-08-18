pipeline {
    agent any  // Puede ser 'any' o un nodo específico

    environment {
        // Variables de entorno globales
        APP_NAME = "MiAplicacion"
        ENV = "development"
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('checkout') {
            steps {
                script {
                    echo "${env.BRANCH_NAME}"
                }
            }
        }

        stage('test') {
            steps {
                npm install
                npm test
            }
        }

                stage('build') {
            steps {
                script {
                    npm install
                    npm build 
                }
            }
        }
    }
}