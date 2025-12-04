pipeline {
    agent any

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
                withCredentials([usernamePassword(credentialsId: 'github-packages-cred', usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                    bat """
                        echo ===== Deploying to GitHub Packages =====

                        mvn -s settings.xml ^
                            -Denv.GH_USER=%GH_USER% ^
                            -Denv.GH_TOKEN=%GH_TOKEN% ^
                            -B deploy
                    """
                }
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
