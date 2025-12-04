pipeline {
    agent any

    environment {
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

        stage('Build & Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'maven-github-settings', variable: 'MAVEN_SETTINGS')]) {
                    bat """
                        echo ======= Setting GitHub Auth ========
                        set GH_USER=%GITHUB_CREDS_USR%
                        set GH_TOKEN=%GITHUB_CREDS_PSW%

                        echo ======= Building Project ===========
                        "%MAVEN_HOME%\\bin\\mvn" -s "%MAVEN_SETTINGS%" -B clean package

                        echo ======= Deploying to GitHub Packages ==
                        "%MAVEN_HOME%\\bin\\mvn" -s "%MAVEN_SETTINGS%" -B deploy
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
