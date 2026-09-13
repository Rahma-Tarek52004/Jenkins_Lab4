
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
                echo "Checking out branch: ${BRANCH_NAME}"

                git branch: "${BRANCH_NAME}",
                    url: 'https://github.com/Rahma-Tarek52004/Jenkins_Lab3.git'
            }
        }

        stage('Check Docker') {
            steps {
                sh '''
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
                    echo "Testing Node.js application..."

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
                    echo "Building Docker image..."

                    DOCKER_BUILDKIT=0 docker build \
                        --no-cache \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                    echo "${DOCKER_CREDENTIALS_PSW}" | docker login \
                        -u "${DOCKER_CREDENTIALS_USR}" \
                        --password-stdin
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    echo "Tagging Docker image..."

                    docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    echo "Pushing Docker image..."

                    docker push ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    echo "Removing old container if it exists..."

                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Starting new Node.js container..."

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 3000:3000 \
                        ${DOCKER_IMAGE}

                    echo "Container started."

                    docker ps
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Waiting for Node.js application to start..."

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
            echo "Branch: ${BRANCH_NAME}"
            echo "Docker Image: ${DOCKER_IMAGE}"
            echo "Application: http://localhost:3000"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "Pipeline failed!"
            echo "Branch: ${BRANCH_NAME}"
            echo "Check the stage that failed above."
            echo "========================================"
        }

        always {
            sh '''
                docker logout || true
            '''
        }
    }
}

