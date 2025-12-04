pipeline {
    agent any

    environment {
        // Jenkins automatically creates these from credentials
        GITHUB_USERNAME = credentials('github-packages-cred').split(':')[0]
        GITHUB_PASSWORD = credentials('github-packages-cred').split(':')[1]
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

        stage('Deploy to GitHub Packages') {
            steps {
                // Create settings.xml dynamically with correct variables
                script {
                    def settingsContent = """
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>github</id>
            <username>${env.GITHUB_USERNAME}</username>
            <password>${env.GITHUB_PASSWORD}</password>
        </server>
    </servers>
</settings>
"""
                    writeFile file: 'deploy-settings.xml', text: settingsContent
                }

                bat """
                    echo "=== Starting Deployment ==="
                    echo "GitHub User: %GITHUB_USERNAME%
                    
                    # First, build the package
                    mvn clean compile package -DskipTests
                    
                    # Deploy with the custom settings file
                    echo "Deploying to GitHub Packages..."
                    mvn deploy -s deploy-settings.xml -DskipTests
                    
                    # Alternative: Direct deploy command
                    mvn deploy:deploy-file \\
                        -Dfile=target/jenkins-demo-1.0.0.jar \\
                        -DpomFile=pom.xml \\
                        -DrepositoryId=github \\
                        -Durl=https://maven.pkg.github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline \\
                        -s deploy-settings.xml
                    
                    echo "=== Deployment Complete ==="
                """
            }
        }
    }

    post {
        always {
            bat 'if exist deploy-settings.xml del deploy-settings.xml'
            bat 'if exist target rmdir /s /q target'
        }
        success {
            echo "✅ Success! Check packages at:"
            echo "https://github.com/hridiklal/Build-and-Deploy-using-maven--Pipeline/packages"
        }
    }
}
