
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        DOCKER = 'C:\\Users\\khadd\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE = 'manar04/monapp'
        TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/khmanar485-create/jenkins.git',
                    branch: 'main'
            }
        }

        stage('Docker Build') {
            steps {
                bat '''
                    "%DOCKER%" build -t "%IMAGE%:%TAG%" .
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                bat '''
                    "%DOCKER%" rm -f monapp-test 2>NUL
                    "%DOCKER%" run -d -p 8081:80 --name monapp-test "%IMAGE%:%TAG%"
                    ping -n 4 127.0.0.1 > nul
                    curl -I http://localhost:8081 | findstr /C:"200 OK"
                    if errorlevel 1 exit /b 1
                    "%DOCKER%" rm -f monapp-test
                '''
            }
        }

        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    bat '''
                        echo %DOCKERHUB_PASS% | "%DOCKER%" login -u "%DOCKERHUB_USER%" --password-stdin
                        "%DOCKER%" tag "%IMAGE%:%TAG%" "%IMAGE%:latest"
                        "%DOCKER%" push "%IMAGE%:%TAG%"
                        "%DOCKER%" push "%IMAGE%:latest"
                        "%DOCKER%" logout
                    '''
                }
            }
        }
    }

    post {
        always {
            bat(returnStatus: true, script: '"%DOCKER%" rm -f monapp-test')
        }
        success {
            echo 'Build OK'
        }
        failure {
            echo 'Build KO'
        }
    }
}
