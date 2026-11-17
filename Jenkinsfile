
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Users\\khadd\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE = 'manar04/monapp'
        TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Verify Files') {
            steps {
                bat 'dir'
            }
        }

        stage('Docker Build') {
            steps {
                bat '"%DOCKER%" build -t "%IMAGE%:%TAG%" .'
            }
        }

        stage('Smoke Test') {
            steps {
                bat '''
                    "%DOCKER%" run -d -p 8081:80 --name monapp-test "%IMAGE%:%TAG%"
                    ping -n 4 127.0.0.1 > nul
                    curl -I http://localhost:8081 | findstr /C:"200 OK"
                    if errorlevel 1 exit /b 1
                    "%DOCKER%" rm -f monapp-test
                '''
            }
        }
    }

    post {
        always {
            bat(returnStatus: true, script: '"%DOCKER%" rm -f monapp-test')
        }
    }
}

