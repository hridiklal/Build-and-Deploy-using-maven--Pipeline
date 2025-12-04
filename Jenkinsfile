pipeline {
    agent any

    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME = tool name: 'jdk11'
        MAVEN_HOME = tool name: 'maven123'
        PATH = "${JAVA_HOME}\\bin;${PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'maven-github-settings', variable: 'MAVEN_SETTINGS')]) {
                    bat """
                        # Set environment variables for Maven settings.xml
                        set GH_USER=%GITHUB_CREDS_USR%
                        set GH_TOKEN=%GITHUB_CREDS_PSW%
                        
                        # Debug information
                        echo "=== Deployment Information ==="
                        echo "GitHub User: %GH_USER%"
                        echo "Settings file: %MAVEN_SETTINGS%"
                        echo "Maven Home: %MAVEN_HOME%"
                        echo "============================="
                        
                        # Test Maven version first
                        "%MAVEN_HOME%\\bin\\mvn" --version
                        
                        # Build with verbose output
                        echo "Step 1: Cleaning and packaging..."
                        "%MAVEN_HOME%\\bin\\mvn" -s "%MAVEN_SETTINGS%" -B clean package -DskipTests
                        
                        # Deploy to GitHub Packages with explicit repository
                        echo "Step 2: Deploying to GitHub Packages..."
                        "%MAVEN_HOME%\\bin\\mvn" -s "%MAVEN_SETTINGS%" -B deploy -DskipTests
                        
                        echo "Deployment command completed."
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment completed successfully."
            echo "Check https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
        }
        failure {
            echo "Pipeline failed. Check console output for details."
        }
        always {
            echo "Pipeline completed. Cleaning up..."
        }
    }
}
