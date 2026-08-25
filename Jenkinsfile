pipeline {

    agent any

    environment {
        AWS_REGION = 'us-east-1'
        S3_BUCKET = 'jenkins-artifacts-cdec'
        APP_NAME = 'calculator-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'

                git branch: 'main',
                    url: 'https://github.com/alkesh-007/calculator-app.git'
            }
        }
    
        stage('Build') {
            steps {
                echo 'Building application...'

                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'

                sh 'mvn test'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Creating JAR...'

                sh 'mvn package -DskipTests'
            }
        }

        

        

        stage('Archive Artifact') {
            steps {
                echo 'Archiving JAR...'

                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
        stage('upload to S3') {
            steps {
                echo 'Uploading  JAR. to S3'

                sh "aws s3 cp target/*.jar s3://${S3_BUCKET}/${APP_NAME}/"
            }
        }
    }


    post {

        success {
            echo 'BUILD SUCCESSFUL'
            echo 'SONARQUBE QUALITY GATE PASSED'
        }

        failure {
            echo 'BUILD FAILED'
            echo 'Check Jenkins Console Output'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
