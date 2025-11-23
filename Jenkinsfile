pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "13.126.30.239"
        APP_DIR = "/home/ubuntu/app"   // CHANGE THIS if needed
        SERVICE_NAME = "spring-app"
        REPO_URL = "https://github.com/Geetha-R-27/jspgram-spring-cicd-main.git"
    }

    tools {
        maven 'maven'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build Spring Boot App') {
            steps {
                echo "Running maven package..."
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Find JAR Name') {
            steps {
                script {
                    JAR_NAME = sh(script: "ls target/*.jar | head -n 1", returnStdout: true).trim()
                    echo "Generated JAR: ${JAR_NAME}"
                }
            }
        }

        stage('Copy JAR To EC2') {
            steps {
                echo "Copying JAR to EC2 Server..."
                sshagent(['ec2-key']) {
                    sh "scp -o StrictHostKeyChecking=no ${JAR_NAME} ${EC2_USER}@${EC2_HOST}:${APP_DIR}/app.jar"
                }
            }
        }

        stage('Restart Application Service') {
            steps {
                echo "Restarting Application..."
                sshagent(['ec2-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'sudo systemctl restart ${SERVICE_NAME}'"
                }
            }
        }

        stage('Verify App Status') {
            steps {
                sshagent(['ec2-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'sudo systemctl status ${SERVICE_NAME} --no-pager'"
                }
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment Successful! Spring Boot app is running."
        }
        failure {
            echo "❌ Deployment Failed. Check logs!"
        }
    }
}
