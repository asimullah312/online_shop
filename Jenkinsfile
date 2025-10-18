pipeline {
    agent { label 'dev' }

    environment {
        IMAGE_NAME = "asimullah312/online-shop"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        GIT_REPO = "https://github.com/asimullah312/online_shop.git"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📦 Cloning repository..."
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GIT_REPO}"]],
                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '.']]])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login & Push to Docker Hub') {
            steps {
                echo "🔑 Logging into Docker Hub and pushing image..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo "🚀 Deploying with Docker Compose..."
                sh """
                    docker-compose pull || true
                    docker-compose up -d --remove-orphans
                """
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Build, push, and deploy completed!"
        }
        failure {
            echo "❌ FAILURE: Something went wrong, check logs."
        }
    }
}
