pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME = "jenkins_nodejs-app"

        DOCKER_CREDENTIALS = credentials('jenkins_docker')

        IMAGE_TAG = "${BUILD_NUMBER}"

        DOCKER_IMAGE = "${DOCKER_CREDENTIALS_USR}/${IMAGE_NAME}:${IMAGE_TAG}"

        CONTAINER_NAME = "nodejs-container"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${git_branch}"

                git branch: "${git_branch}",
                    url: 'https://github.com/Rahma-Tarek52004/Jenkins_Lab4.git'
            }
        }

        stage('Check Docker') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Checking Docker"
                    echo "========================================"

                    echo "Docker version:"
                    docker version

                    echo ""
                    echo "Docker info:"
                    docker info
                '''
            }
        }

        stage('Test Node.js') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Testing Node.js application"
                    echo "========================================"

                    docker run --rm \
                        -v "$(pwd):/app" \
                        -w /app \
                        node:22-slim \
                        node --check index.js

                    echo "Node.js syntax check passed!"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Building Docker image"
                    echo "========================================"

                    DOCKER_BUILDKIT=0 docker build \
                        --no-cache \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    echo "Docker image built successfully!"
                '''
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Logging in to Docker Hub"
                    echo "========================================"

                    echo "${DOCKER_CREDENTIALS_PSW}" | docker login \
                        -u "${DOCKER_CREDENTIALS_USR}" \
                        --password-stdin

                    echo "Docker login successful!"
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Tagging Docker image"
                    echo "========================================"

                    docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}

                    echo "Image tagged as: ${DOCKER_IMAGE}"
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Pushing Docker image"
                    echo "========================================"

                    docker push ${DOCKER_IMAGE}

                    echo "Docker image pushed successfully!"
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Running Node.js container"
                    echo "========================================"

                    echo "Removing old container if it exists..."

                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Starting new Node.js container..."

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 3000:3000 \
                        ${DOCKER_IMAGE}

                    echo "Container started successfully."

                    echo ""
                    echo "Running containers:"
                    docker ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Health Check"
                    echo "========================================"

                    echo "Waiting for application to start..."

                    sleep 5

                    echo "Checking application health..."

                    curl -f http://localhost:3000/health

                    echo ""
                    echo "Node.js application is healthy!"
                '''
            }
        }
    }

    post {

        success {
            echo "========================================"
            echo "Pipeline completed successfully!"
            echo "========================================"
            echo "Branch: ${git_branch}"
            echo "Docker Image: ${DOCKER_IMAGE}"
            echo "Application: http://localhost:3000"
            echo "Health Check: http://localhost:3000/health"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "Pipeline failed!"
            echo "========================================"
            echo "Branch: ${git_branch}"
            echo "Check the failed stage above."
            echo "========================================"
        }

        always {
            sh '''
                docker logout || true
            '''
        }
    }
}
