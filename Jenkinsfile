pipeline {
    agent any

    environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials') // Replace with your Docker Hub credentials ID
        IMAGE_TAG = "boudhraadhia7/miniproject01:${env.BUILD_ID}" // Replace with your image tag
    }

    stages {
        stage('Checkout') {
            steps {
                // Get the latest code from the SCM
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Build the Docker image for the React app
                script {
                    docker.build("$IMAGE_TAG-client", "-f ./client/Dockerfile ./client")
                }
            }
        }

        stage('Test') {
            steps {
                // Run tests here or any other commands you need
                sh 'echo "Running tests"'
                // sh 'npm test' // Example for Node.js projects
            }
        }

        stage('Docker Push') {
            steps {
                // Push the image to Docker Hub
                script {
                    docker.withRegistry('', "$DOCKERHUB_CREDENTIALS") {
                        docker.image("$IMAGE_TAG-client").push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                // Deploy to Kubernetes
                script {
                    // Set up kubectl
                    // Use 'kubectl' or 'helm' or any other deployment tool you have configured
                    sh 'echo "Deploying to Kubernetes"'
                    sh 'kubectl set image deployment/your-app-deployment your-container=$IMAGE_TAG-client'
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after the build is done
            cleanWs()
        }

        success {
            // Actions to perform on successful build
            sh 'echo "Build succeeded!"'
        }

        failure {
            // Actions to perform if the build fails
            sh 'echo "Build failed!"'
        }
    }
}
