
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'yuvadhanvanth/oltrs'
        TOMCAT_WEBAPPS = 'C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Archive WAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.war',
                                 fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                bat '''
                copy /Y "target\\ticket-reservation.war" "%TOMCAT_WEBAPPS%\\ticket-reservation.war"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    docker login -u "%DOCKER_USER%" --password-stdin < nul
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'OLTRS pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
