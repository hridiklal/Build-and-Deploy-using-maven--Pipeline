pipeline {
    agent any

    environment {
        // Use credentials directly - Jenkins automatically creates GITHUB_CREDS_USR and GITHUB_CREDS_PSW
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME = tool name: 'jdk11'
        MAVEN_HOME = tool name: 'maven123'
        PATH = "${JAVA_HOME}\\bin;${MAVEN_HOME}\\bin;${PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Debug Info') {
            steps {
                bat """
                    echo "=== DEBUG INFORMATION ==="
                    echo "GitHub Credentials USR: %GITHUB_CREDS_USR%"
                    echo "Maven Home: %MAVEN_HOME%"
                    echo "Java Home: %JAVA_HOME%"
                    echo "Current directory:"
                    cd
                    echo "Files in directory:"
                    dir
                    echo "========================="
                """
            }
        }

        stage('Create Settings File') {
            steps {
                script {
                    // Read credentials safely
                    withCredentials([usernamePassword(
                        credentialsId: 'github-packages-cred',
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )]) {
                        def settingsContent = """<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>github</id>
            <username>${GH_USER}</username>
            <password>${GH_TOKEN}</password>
        </server>
    </servers>
</settings>"""
                        writeFile file: 'settings.xml', text: settingsContent
                    }
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-packages-cred',
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )]) {
                        bat """
                            echo "=== STARTING DEPLOYMENT ==="
                            echo "GitHub User: %GH_USER%
                            echo "Using settings.xml:"
                            type settings.xml
                            echo.
                            
                            echo "Step 1: Cleaning..."
                            "%MAVEN_HOME%\\bin\\mvn" clean -DskipTests
                            
                            echo "Step 2: Compiling..."
                            "%MAVEN_HOME%\\bin\\mvn" compile -DskipTests
                            
                            echo "Step 3: Packaging..."
                            "%MAVEN_HOME%\\bin\\mvn" package -DskipTests
                            
                            echo "Step 4: Deploying to GitHub Packages..."
                            "%MAVEN_HOME%\\bin\\mvn" deploy -s settings.xml -DskipTests
                            
                            echo "=== DEPLOYMENT COMPLETE ==="
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            bat 'if exist settings.xml del settings.xml'
            bat 'if exist target rmdir /s /q target'
        }
        success {
            echo "✅ SUCCESS! Build completed."
            echo "📦 Check your GitHub Packages at:"
            echo "https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
        }
        failure {
            echo "❌ FAILURE! Check the console output above."
        }
    }
}
