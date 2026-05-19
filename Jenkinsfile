pipeline {
    agent any

    environment {
        PROJECT = "WebApplication1.csproj"
        APP_POOL = "WebJenkins"
        PUBLISH_DIR = "publish"
        IIS_DIR = "D:\\Backup\\Jenkins\\Publish"
        BACKUP_DIR = "D:\\Backup\\Jenkins\\Publish_backup"
        STAGING_DIR = "D:\\Backup\\Jenkins\\Publish_staging"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                bat 'dotnet test'
            }
        }

        stage('Publish') {
            steps {
                bat """
                    if exist %PUBLISH_DIR% rmdir /s /q %PUBLISH_DIR%
                    mkdir %PUBLISH_DIR%
                    dotnet publish %PROJECT% -c Release -o %PUBLISH_DIR%
                """
            }
        }

        stage('Deploy to IIS') {
            steps {
                bat """
                    echo ===============================
                    echo STOPPING IIS APP POOL
                    echo ===============================

                    powershell -Command "Stop-WebAppPool -Name '%APP_POOL%' -ErrorAction SilentlyContinue"
                    powershell -Command "Stop-Website -Name '%APP_POOL%' -ErrorAction SilentlyContinue"

                    timeout /t 5

                    echo Killing locked dotnet processes...
                    taskkill /F /IM dotnet.exe /T

                    echo ===============================
                    echo BACKING UP OLD DEPLOYMENT
                    echo ===============================

                    if exist "%IIS_DIR%" (
                        robocopy "%IIS_DIR%" "%BACKUP_DIR%" /MIR /NFL /NDL /NJH /NJS
                    )

                    echo ===============================
                    echo CLEANING OLD FILES
                    echo ===============================

                    powershell -Command "if (Test-Path '%IIS_DIR%') { Remove-Item -Recurse -Force '%IIS_DIR%' }"

                    mkdir "%STAGING_DIR%"

                    echo ===============================
                    echo COPYING NEW BUILD
                    echo ===============================

                    robocopy "%PUBLISH_DIR%" "%STAGING_DIR%" /MIR /NFL /NDL /NJH /NJS

                    move "%STAGING_DIR%" "%IIS_DIR%"

                    echo ===============================
                    echo STARTING IIS APP POOL
                    echo ===============================

                    timeout /t 5

                    powershell -Command "Start-WebAppPool -Name '%APP_POOL%'"
                    powershell -Command "Start-Website -Name '%APP_POOL%'"
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS ✅ Application deployed successfully"
        }

        failure {
            echo "Pipeline FAILED ❌ Check logs"
        }

        always {
            cleanWs()
            echo "Workspace cleaned"
        }
    }
}