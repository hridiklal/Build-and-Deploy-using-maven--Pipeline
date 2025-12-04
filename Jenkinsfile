pipeline {
    agent any

    environment {
        GH_USER = credentials('github-username')
        GH_TOKEN = credentials('github-token')
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
                    mvn -B clean package
                """
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                bat """
                    mvn -s settings.xml -B deploy
                """
            }
        }
    }

    post {
        success {
            echo "Build & Deploy completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
