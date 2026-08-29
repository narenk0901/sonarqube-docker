pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat 'python -m pip install -r requirements.txt'
                bat 'python -m pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool 'SonarQubeScanner'
                        bat """
                            "${scannerHome}\\bin\\sonar-scanner.bat" ^
                            -Dsonar.projectKey=sonarqube-docker ^
                            -Dsonar.projectName=sonarqube-docker ^
                            -Dsonar.sources=. ^
                            -Dsonar.exclusions=**/test_*.py
                        """
                    }
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