pipeline {
    agent any          
 
    tools {
        nodejs "NodeJS_7.8"
    }
 
    triggers {
        pollSCM('* * * * *') // cad 1 minuto se fija si hy cambios en las ramas?
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
                bat 'npm config set strict-ssl false'
                bat 'npm install'
                bat 'npm test'
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
       
       /*  stage('build docker image') {
            steps {
                echo 'Deploying...'
                // commands to deploy your app
            }
        }
 
        stage('deploy') {
            steps {
                echo 'Deploying...'
                // commands to deploy your app
            }
        } */
    }
}