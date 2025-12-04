pipeline {
    agent any

    tools {
        maven 'maven123'    // This makes Jenkins use your installed Maven tool
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Project') {
            steps {
                script {
                    def mvnHome = tool 'maven123'
                    bat """
                        echo ===== Building =====
                        "${mvnHome}\\bin\\mvn" -B clean package
                    """
                }
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-packages-cred', usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                    script {
                        def mvnHome = tool 'maven123'
                        bat """
                            echo ===== Deploying =====
                            "${mvnHome}\\bin\\mvn" -s settings.xml ^
                                -Denv.GH_USER=%GH_USER% ^
                                -Denv.GH_TOKEN=%GH_TOKEN% ^
                                -B deploy
                        """
                    }
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
