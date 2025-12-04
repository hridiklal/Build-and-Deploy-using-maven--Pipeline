pipeline {
    agent any

    environment {
        GH_USER = credentials('github-packages-cred').usr
        GH_TOKEN = credentials('github-packages-cred').psw
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Project') {
            steps {
                bat """
                    echo ===== Building =====
                    mvn -B clean package
                """
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                bat """
                    echo ===== Deploying to GitHub Packages =====
                    mvn -s settings.xml -DskipTests=false -B deploy
                """
            }
        }
    }

    post {
        success {
            echo "Build & Deployment completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
