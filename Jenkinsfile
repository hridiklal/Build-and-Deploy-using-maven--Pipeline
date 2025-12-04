pipeline {
    agent any

    stages {
        stage('Setup Environment') {
            steps {
                bat '''
                    echo ===========================================
                    echo STEP 1: SETUP ENVIRONMENT
                    echo ===========================================
                    
                    echo Current directory:
                    cd
                    echo.
                    
                    echo Checking Java installation:
                    java -version
                    echo.
                    
                    echo Listing current files:
                    dir
                    echo.
                '''
            }
        }
        
        stage('Install Maven') {
            steps {
                bat '''
                    echo ===========================================
                    echo STEP 2: INSTALL MAVEN
                    echo ===========================================
                    
                    echo Downloading Apache Maven 3.9.9...
                    curl -L -o maven.zip https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
                    
                    echo Checking download...
                    dir maven.zip
                    echo.
                    
                    echo Extracting Maven...
                    powershell -Command "Expand-Archive -Path 'maven.zip' -DestinationPath '.' -Force"
                    
                    echo Setting MAVEN_HOME...
                    set MAVEN_HOME=%cd%\\apache-maven-3.9.9
                    echo MAVEN_HOME = %MAVEN_HOME%
                    
                    echo Verifying Maven installation...
                    "%MAVEN_HOME%\\bin\\mvn.cmd" --version
                    echo.
                '''
            }
        }
        
        stage('Build Project') {
            steps {
                bat '''
                    echo ===========================================
                    echo STEP 3: BUILD PROJECT
                    echo ===========================================
                    
                    set MAVEN_HOME=%cd%\\apache-maven-3.9.9
                    
                    echo Checking pom.xml...
                    if exist pom.xml (
                        echo Found pom.xml
                        echo First few lines:
                        head -n 10 pom.xml
                    ) else (
                        echo ERROR: pom.xml not found!
                        exit 1
                    )
                    echo.
                    
                    echo Cleaning project...
                    "%MAVEN_HOME%\\bin\\mvn.cmd" clean
                    echo Return code: %ERRORLEVEL%
                    echo.
                    
                    echo Compiling project...
                    "%MAVEN_HOME%\\bin\\mvn.cmd" compile
                    echo Return code: %ERRORLEVEL%
                    echo.
                    
                    echo Creating JAR file...
                    "%MAVEN_HOME%\\bin\\mvn.cmd" package -DskipTests
                    echo Return code: %ERRORLEVEL%
                    echo.
                    
                    echo Checking if JAR was created...
                    dir target\\*.jar || echo No JAR files found
                    echo.
                '''
            }
        }
        
        stage('Deploy to GitHub Packages') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-packages-cred',
                    usernameVariable: 'GITHUB_USERNAME',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {
                    bat """
                        echo ===========================================
                        echo STEP 4: DEPLOY TO GITHUB PACKAGES
                        echo ===========================================
                        
                        set MAVEN_HOME=%cd%\\apache-maven-3.9.9
                        
                        echo GitHub Username: %GITHUB_USERNAME%
                        echo.
                        
                        echo Creating Maven settings.xml with GitHub credentials...
                        echo ^<?xml version="1.0" encoding="UTF-8"?^> > settings.xml
                        echo ^<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"^> >> settings.xml
                        echo   ^<servers^> >> settings.xml
                        echo     ^<server^> >> settings.xml
                        echo       ^<id^>github^</id^> >> settings.xml
                        echo       ^<username^>%GITHUB_USERNAME%^</username^> >> settings.xml
                        echo       ^<password^>%GITHUB_TOKEN%^</password^> >> settings.xml
                        echo     ^</server^> >> settings.xml
                        echo   ^</servers^> >> settings.xml
                        echo ^</settings^> >> settings.xml
                        
                        echo Settings file created. Contents:
                        type settings.xml
                        echo.
                        
                        echo ===== STARTING DEPLOYMENT =====
                        echo This will upload your package to GitHub Packages...
                        echo.
                        
                        echo Deploying with Maven...
                        "%MAVEN_HOME%\\bin\\mvn.cmd" deploy -s settings.xml -DskipTests
                        
                        echo Deployment command completed.
                        echo Return code: %ERRORLEVEL%
                        echo.
                        
                        echo Cleaning up settings.xml...
                        del settings.xml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "==========================================="
            echo "✅ PIPELINE SUCCESSFUL!"
            echo "==========================================="
            echo ""
            echo "📦 **YOUR PACKAGE SHOULD NOW BE IN GITHUB!**"
            echo ""
            echo "👉 Check here:"
            echo "https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
            echo ""
            echo "⏱️  It may take 1-2 minutes to appear"
            echo "🔄 Refresh the page after a minute"
            echo ""
            echo "💡 If you don't see it, check:"
            echo "1. GitHub token has 'write:packages' permission"
            echo "2. Repository URL in pom.xml is correct"
            echo "3. Check Jenkins console output for 'Uploading to github:'"
            echo "==========================================="
        }
        failure {
            echo "❌ PIPELINE FAILED"
            echo "Check the console output above for specific errors"
        }
        always {
            bat '''
                echo ===========================================
                echo CLEANING UP TEMPORARY FILES
                echo ===========================================
                if exist maven.zip del maven.zip
                if exist apache-maven-3.9.9 rmdir /s /q apache-maven-3.9.9
                echo Cleanup complete.
            '''
        }
    }
}
