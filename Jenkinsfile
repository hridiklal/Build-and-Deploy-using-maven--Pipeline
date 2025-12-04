pipeline {
    agent any

    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME = tool name: 'jdk11'
        MAVEN_HOME = tool name: 'maven123'
        PATH = "${JAVA_HOME}\\bin;${MAVEN_HOME}\\bin;${PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                script {
                    // Use bat script to handle everything in one go
                    bat """
                        echo ========================================
                        echo DEPLOYING TO GITHUB PACKAGES
                        echo ========================================
                        
                        REM Create a simple settings.xml with GitHub credentials
                        echo ^<?xml version="1.0" encoding="UTF-8"?^> > deploy-settings.xml
                        echo ^<settings^> >> deploy-settings.xml
                        echo   ^<servers^> >> deploy-settings.xml
                        echo     ^<server^> >> deploy-settings.xml
                        echo       ^<id^>github^</id^> >> deploy-settings.xml
                        echo       ^<username^>%GITHUB_CREDS_USR%^</username^> >> deploy-settings.xml
                        echo       ^<password^>%GITHUB_CREDS_PSW%^</password^> >> deploy-settings.xml
                        echo     ^</server^> >> deploy-settings.xml
                        echo   ^</servers^> >> deploy-settings.xml
                        echo ^</settings^> >> deploy-settings.xml
                        
                        echo ========================================
                        echo 1. DISPLAYING CREDENTIALS INFO
                        echo ========================================
                        echo GitHub Username: %GITHUB_CREDS_USR%
                        
                        echo ========================================
                        echo 2. BUILDING PROJECT
                        echo ========================================
                        "%MAVEN_HOME%\\bin\\mvn" clean compile -DskipTests
                        
                        echo ========================================
                        echo 3. PACKAGING
                        echo ========================================
                        "%MAVEN_HOME%\\bin\\mvn" package -DskipTests
                        
                        echo ========================================
                        echo 4. DEPLOYING TO GITHUB PACKAGES
                        echo ========================================
                        echo Deploying jar file to GitHub...
                        "%MAVEN_HOME%\\bin\\mvn" deploy:deploy-file ^
                          -Dfile=target/jenkins-demo-1.0.0.jar ^
                          -DpomFile=pom.xml ^
                          -DrepositoryId=github ^
                          -Durl=https://maven.pkg.github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline ^
                          -s deploy-settings.xml ^
                          -DskipTests
                        
                        echo ========================================
                        echo 5. VERIFYING DEPLOYMENT
                        echo ========================================
                        if exist target/jenkins-demo-1.0.0.jar (
                            echo JAR file exists: target/jenkins-demo-1.0.0.jar
                            echo Size: 
                            for %%F in ("target/jenkins-demo-1.0.0.jar") do echo %%~zF bytes
                        ) else (
                            echo ERROR: JAR file not found!
                        )
                        
                        echo ========================================
                        echo CLEANUP
                        echo ========================================
                        del deploy-settings.xml
                        
                        echo ========================================
                        echo DEPLOYMENT PROCESS COMPLETED
                        echo ========================================
                        echo.
                        echo NEXT STEPS:
                        echo 1. Visit: https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages
                        echo 2. Refresh the page in 1-2 minutes
                        echo 3. You should see your package
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ BUILD SUCCESSFUL!"
            echo "📦 Check your GitHub Packages:"
            echo "https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
            echo ""
            echo "⚠️  Note: It may take 1-2 minutes for packages to appear"
        }
        failure {
            echo "❌ BUILD FAILED"
            echo "Check console output above for errors"
        }
    }
}
