pipeline {
    agent any

    environment {
        PROJECT = "WebApplication1.csproj"
        PUBLISH_DIR = "publish"
        IIS_PATH = "D:\\Backup\\Jenkins\\Publish"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                url: 'https://github.com/muniyan26/WebAppJenkins.git'
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
                bat "dotnet publish %PROJECT% -c Release -o %PUBLISH_DIR%"
            }
        }

        stage('Deploy to IIS') {
            steps {
                bat """
                powershell Stop-WebAppPool -Name 'MyAppPool'
                xcopy /E /Y publish\\* %IIS_PATH%
                powershell Start-WebAppPool -Name 'MyAppPool'
                """
            }
        }
    }
}