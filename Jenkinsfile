pipeline {
    agent any
    
    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
    }

    stages {
        stage('System Information') {
            steps {
                bat '''
                    echo ===========================================
                    echo SYSTEM DIAGNOSTICS
                    echo ===========================================
                    echo Date: %DATE% %TIME%
                    echo.
                    
                    echo 1. JAVA VERSION:
                    java -version
                    echo.
                    
                    echo 2. MAVEN VERSION:
                    call mvn --version
                    echo.
                    
                    echo 3. CURRENT DIRECTORY:
                    cd
                    echo.
                    
                    echo 4. DIRECTORY CONTENTS:
                    dir
                    echo.
                    
                    echo 5. GITHUB CREDENTIALS AVAILABLE:
                    echo Username variable exists: %GITHUB_CREDS_USR%
                    echo Password variable exists: %GITHUB_CREDS_PSW%
                    echo.
                '''
            }
        }
        
        stage('Check Project Structure') {
            steps {
                bat '''
                    echo ===========================================
                    echo PROJECT STRUCTURE CHECK
                    echo ===========================================
                    
                    echo 1. POM.XML CONTENTS:
                    type pom.xml
                    echo.
                    
                    echo 2. CHECK IF JAR IS CREATED:
                    call mvn clean package -DskipTests -q
                    echo.
                    
                    echo 3. VERIFY TARGET FOLDER:
                    if exist target (
                        echo Target folder exists
                        dir target
                        echo.
                        echo JAR file details:
                        for %%F in ("target\\*.jar") do (
                            echo Found: %%~nF%%~xF (%%~zF bytes)
                        )
                    ) else (
                        echo ERROR: Target folder not created!
                    )
                    echo.
                '''
            }
        }
        
        stage('Test GitHub Connection') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-packages-cred',
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )]) {
                        bat """
                            echo ===========================================
                            echo GITHUB CONNECTION TEST
                            echo ===========================================
                            
                            echo 1. TESTING GITHUB API ACCESS:
                            echo Testing with username: %GH_USER%
                            echo.
                            
                            echo 2. CREATE TEST SETTINGS.XML:
                            echo ^<?xml version="1.0" encoding="UTF-8"?^> > test-settings.xml
                            echo ^<settings^> >> test-settings.xml
                            echo   ^<servers^> >> test-settings.xml
                            echo     ^<server^> >> test-settings.xml
                            echo       ^<id^>github^</id^> >> test-settings.xml
                            echo       ^<username^>%GH_USER%^</username^> >> test-settings.xml
                            echo       ^<password^>%GH_TOKEN%^</password^> >> test-settings.xml
                            echo     ^</server^> >> test-settings.xml
                            echo   ^</servers^> >> test-settings.xml
                            echo ^</settings^> >> test-settings.xml
                            echo.
                            
                            echo 3. SETTINGS.XML CONTENTS:
                            type test-settings.xml
                            echo.
                            
                            echo 4. TEST MAVEN DEPLOY COMMAND (Dry Run):
                            call mvn deploy -s test-settings.xml -DskipTests -DdryRun=true 2>&1 | findstr /i "upload deploy"
                            echo.
                            
                            echo 5. TEST DIRECT UPLOAD:
                            echo Testing deploy:deploy-file command...
                            call mvn deploy:deploy-file ^
                              -Dfile=target/jenkins-demo-1.0.0.jar ^
                              -DpomFile=pom.xml ^
                              -DrepositoryId=github ^
                              -Durl=https://maven.pkg.github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline ^
                              -s test-settings.xml ^
                              -DskipTests 2>&1 | findstr /i "error fail success"
                            echo.
                            
                            del test-settings.xml
                        """
                    }
                }
            }
        }
        
        stage('Full Deploy Test') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-packages-cred',
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )]) {
                        bat """
                            echo ===========================================
                            echo FULL DEPLOYMENT TEST WITH VERBOSE OUTPUT
                            echo ===========================================
                            
                            echo Creating settings.xml...
                            echo ^<?xml version="1.0" encoding="UTF-8"?^> > deploy-settings.xml
                            (
                            echo ^<settings^>
                            echo   ^<servers^>
                            echo     ^<server^>
                            echo       ^<id^>github^</id^>
                            echo       ^<username^>%GH_USER%^</username^>
                            echo       ^<password^>%GH_TOKEN%^</password^>
                            echo     ^</server^>
                            echo   ^</servers^>
                            echo ^</settings^>
                            ) >> deploy-settings.xml
                            
                            echo.
                            echo ===== STEP 1: CLEAN AND BUILD =====
                            call mvn clean compile -DskipTests
                            echo.
                            
                            echo ===== STEP 2: PACKAGE =====
                            call mvn package -DskipTests
                            echo.
                            
                            echo ===== STEP 3: DEPLOY (WITH FULL OUTPUT) =====
                            echo Starting deploy command...
                            call mvn deploy -s deploy-settings.xml -DskipTests -X
                            echo.
                            
                            echo ===== STEP 4: CHECK FOR SUCCESS =====
                            echo Checking console output for upload indicators...
                            echo If you see "Uploading to github:" above, then deploy worked.
                            echo.
                            
                            del deploy-settings.xml
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "==========================================="
            echo "DIAGNOSTIC COMPLETE"
            echo "==========================================="
            echo "Check the console output above for:"
            echo "1. 'Uploading to github:' - Means package was sent"
            echo "2. 'BUILD SUCCESS' - Means Maven succeeded"
            echo "3. Any ERROR messages"
            echo ""
            echo "After running, check your GitHub Packages page:"
            echo "https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
            echo "==========================================="
        }
    }
}
