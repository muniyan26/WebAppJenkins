pipeline {
    agent any

    parameters {
        booleanParam(name: 'ROLLBACK', defaultValue: false, description: 'Rollback to previous version')
    }

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
                if exist %PUBLISH_DIR% rmdir /s /q %PUBLISH_DIR%
                mkdir %PUBLISH_DIR%

                dotnet clean %PROJECT%
                dotnet publish %PROJECT% -c Release -o %PUBLISH_DIR%
                """
            }
        }

        stage('Deploy to IIS (Safe)') {
            when {
                expression { return params.ROLLBACK == false }
            }
            steps {
                bat """
                set BACKUP_DIR=%IIS_PATH%_backup
                set STAGING_DIR=%IIS_PATH%_staging

                echo Stopping IIS App Pool...
                powershell -Command "Stop-WebAppPool -Name 'WebJenkins'"

                echo Creating backup...
                if exist %IIS_PATH% (
                    robocopy %IIS_PATH% %BACKUP_DIR% /MIR /NFL /NDL /NJH /NJS
                )

                echo Preparing staging folder...
                if exist %STAGING_DIR% rmdir /s /q %STAGING_DIR%
                mkdir %STAGING_DIR%

                echo Copying publish output to staging...
                robocopy %PUBLISH_DIR% %STAGING_DIR% /MIR /NFL /NDL /NJH /NJS

                echo Replacing IIS folder with new version...
                rmdir /s /q %IIS_PATH%
                move %STAGING_DIR% %IIS_PATH%

                echo Starting IIS App Pool...
                powershell -Command "Start-WebAppPool -Name 'WebJenkins'"
                """
            }
        }

        stage('Rollback') {
            when {
                expression { return params.ROLLBACK == true }
            }
            steps {
                bat """
                echo ROLLBACK INITIATED

                powershell -Command "Stop-WebAppPool -Name 'WebJenkins'"

                if exist %IIS_PATH% rmdir /s /q %IIS_PATH%

                robocopy %IIS_PATH%_backup %IIS_PATH% /MIR /NFL /NDL /NJH /NJS

                powershell -Command "Start-WebAppPool -Name 'WebJenkins'"

                echo Rollback completed successfully
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully 🎉'
        }

        failure {
            echo 'Pipeline failed ❌ Check logs'
        }

        unstable {
            echo 'Build unstable ⚠️'
        }

        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}