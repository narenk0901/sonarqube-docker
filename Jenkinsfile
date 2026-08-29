pipeline {
    agent any

    tools {
        sonarQube 'SonarQubeScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat 'pip install -r requirements.txt'
                bat 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat '''
                    sonar-scanner ^
                    -Dsonar.projectKey=sonarqube-docker ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://localhost:9000
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t sonarqube-docker-app .'
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker rm -f sonarqube-app || exit 0'
                bat 'docker run -d --name sonarqube-app sonarqube-docker-app'
            }
        }
    }
}