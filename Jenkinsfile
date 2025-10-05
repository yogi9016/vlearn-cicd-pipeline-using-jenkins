pipeline {
    agent any

    environment {
        APP_NAME = "flask-cicd-demo"
        GIT_REPO = "https://github.com/<your-username>/python-sample-flask.git"
        DOCKER_IMAGE = "flask-cicd-demo:latest"
        CONTAINER_NAME = "flask-cicd-container"
        APP_PORT = "5000"
    }

    stages {

        stage('Check Dependencies') {
            steps {
                echo 'Checking required dependencies...'

                // Check Docker
                sh '''
                if ! command -v docker &> /dev/null; then
                    echo "Docker not found. Installing Docker..."
                    sudo apt-get update && sudo apt-get install -y docker.io
                else
                    echo "Docker is already installed."
                fi

                // Check NGINX
                if ! command -v nginx &> /dev/null; then
                    echo "NGINX not found. Installing NGINX..."
                    sudo apt-get install -y nginx
                else
                    echo "NGINX is already installed."
                fi

                // Check Git
                if ! command -v git &> /dev/null; then
                    echo "Git not found. Installing Git..."
                    sudo apt-get install -y git
                else
                    echo "Git is already installed."
                fi

                echo "Checking services..."
                sudo systemctl is-active --quiet docker && echo "Docker running ✅" || echo "Docker not running ❌"
                sudo systemctl is-active --quiet nginx && echo "NGINX running ✅" || echo "NGINX not running ❌"
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo 'Cloning the repository...'
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                docker build -t ${DOCKER_IMAGE} .
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting the Flask application container...'
                sh '''
                # Stop old container if running
                if [ "$(docker ps -q -f name=${CONTAINER_NAME})" ]; then
                    docker stop ${CONTAINER_NAME}
                    docker rm ${CONTAINER_NAME}
                fi

                docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Performing health check...'
                script {
                    def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${APP_PORT}/health", returnStdout: true).trim()
                    if (status == "200") {
                        echo "Health check passed ✅"
                    } else {
                        error("Health check failed ❌ - Status code: ${status}")
                    }
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ Jenkins Build Succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "The Flask CI/CD pipeline completed successfully!",
                to: "team@example.com"
            )
        }
        failure {
            emailext (
                subject: "❌ Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "The Flask CI/CD pipeline failed. Check Jenkins logs.",
                to: "team@example.com"
            )
        }
    }
}