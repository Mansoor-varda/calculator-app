pipeline {

    agent any

    environment {
        SONAR_PROJECT_KEY = 'calculator-app'
        aws_region = 'us-east-1'
        s3_bucket = 's3-tfstate-file-test'
        app_name = 'calculator-app'
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

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis..'

                withSonarQubeEnv('SonarQube') {
                sh '''
                  mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                  -Dsonar.projectKey=calculator-app \
                  -Dsonar.projectName=calculator-app
                  '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate..'

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving JAR...'

                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }


        stage('Upload to S3') {
            steps {
                echo 'Uploading JAR to S3...'

                sh '''
                  aws s3 cp target/*.jar s3://$s3_bucket/$app_name/ --region $aws_region)
                  '''
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