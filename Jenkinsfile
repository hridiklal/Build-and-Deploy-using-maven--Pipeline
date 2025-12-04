pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Maven') {
            steps {
                bat '''
                    echo ===========================================
                    echo INSTALLING MAVEN ON THE FLY
                    echo ===========================================
                    
                    REM Check if Maven exists
                    where mvn > nul 2>&1
                    if %ERRORLEVEL% EQU 0 (
                        echo Maven is already installed
                        mvn --version
                    ) else (
                        echo Installing Maven...
                        
                        REM Download Maven
                        curl -L -o apache-maven.zip https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
                        
                        REM Extract Maven
                        powershell -Command "Expand-Archive -Path apache-maven.zip -DestinationPath ."
                        
                        REM Set Maven path
                        set MAVEN_HOME=%cd%\apache-maven-3.9.9
                        set PATH=%MAVEN_HOME%\bin;%PATH%
                        
                        echo Maven installed at: %MAVEN_HOME%
                        mvn --version
                    )
                    echo.
                '''
            }
        }

        stage('Build & Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-packages-cred',
                    usernameVariable: 'GITHUB_USER',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {
                    bat '''
                        echo ===========================================
                        echo BUILDING AND DEPLOYING
                        echo ===========================================
                        
                        REM Set Maven path again (in case of new shell)
                        set MAVEN_HOME=%cd%\apache-maven-3.9.9
                        set PATH=%MAVEN_HOME%\bin;%PATH%
                        
                        echo 1. Creating settings.xml...
                        (
                        echo ^<?xml version="1.0" encoding="UTF-8"?^>
                        echo ^<settings^>
                        echo   ^<servers^>
                        echo     ^<server^>
                        echo       ^<id^>github^</id^>
                        echo       ^<username^>%GITHUB_USER%^</username^>
                        echo       ^<password^>%GITHUB_TOKEN%^</password^>
                        echo     ^</server^>
                        echo   ^</servers^>
                        echo ^</settings^>
                        ) > deploy-settings.xml
                        
                        echo 2. Building project...
                        call mvn clean compile -DskipTests
                        
                        echo 3. Creating JAR file...
                        call mvn package -DskipTests
                        
                        echo 4. Checking JAR file...
                        dir target\\*.jar
                        
                        echo 5. DEPLOYING TO GITHUB PACKAGES...
                        echo This may take a moment...
                        call mvn deploy -s deploy-settings.xml -DskipTests
                        
                        echo 6. Verifying deployment...
                        echo If successful, you should see "Uploading to github:" above
                        
                        REM Cleanup
                        del deploy-settings.xml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ BUILD SUCCESSFUL!"
            echo ""
            echo "📦 **IMPORTANT: Check your GitHub Packages now:**"
            echo "👉 https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
            echo ""
            echo "⏱️  It may take 1-2 minutes to appear"
            echo "🔄 Refresh the page if you don't see it immediately"
        }
        failure {
            echo "❌ BUILD FAILED"
            echo "Check the console output above for specific errors"
        }
        always {
            bat '''
                echo Cleaning up temporary files...
                if exist apache-maven.zip del apache-maven.zip
                if exist apache-maven-3.9.9 rmdir /s /q apache-maven-3.9.9
            '''
        }
    }
}
