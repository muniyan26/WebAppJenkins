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

                bat """
                del /f /s /q "%PUBLISH_DIR%\\*"
                for /d %%p in ("%PUBLISH_DIR%\\*") do rmdir "%%p" /s /q
                dotnet clean %PROJECT%
                dotnet publish %PROJECT% -c Release -o "%PUBLISH_DIR%"
                """
                //bat "dotnet publish %PROJECT% -c Release -o %PUBLISH_DIR%"
            }
        }

        stage('Deploy to IIS') {
            steps {
                bat """
                powershell Stop-WebAppPool -Name 'WebJenkins'
                robocopy publish %IIS_PATH% /MIR /NFL /NDL /NJH /NJS
                powershell Start-WebAppPool -Name 'WebJenkins'
                """
            }
        }
    }
}