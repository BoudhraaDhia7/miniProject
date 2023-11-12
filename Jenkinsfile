pipeline {
    agent any

    environment {
        // Define environment variables, Docker registry, etc.
        DOCKER_IMAGE_BACKEND  = "/server"
        DOCKER_IMAGE_FRONTEND = "/client"
        // More environment variables can be added here
    }

    stages {
        stage('Checkout') {
            steps {
                // Get the latest code from your source control
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    // Build the Docker image for the backend
                    docker.build "${DOCKER_IMAGE_BACKEND}" , 
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    // Build the Docker image for the frontend
                    docker.build "${DOCKER_IMAGE_FRONTEND}", 
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Push to Registry') {
        when {
            branch 'main' // Only push images for the main branch
        }
        steps {
            script {
                // Login to Docker Hub and push the images
                docker.withRegistry('https://registry.hub.docker.com', 'boudhraadhia7') {
                    docker.image("${DOCKER_IMAGE_BACKEND}").push()
                    docker.image("${DOCKER_IMAGE_FRONTEND}").push()
                }
            }
        }
    }

        stage('Deploy') {
            when {
                branch 'main' 
            }
            steps {
                echo 'Deployment steps will go here'
            }
        }
    }

    post {
        always {
            echo 'Performing some actions Als ist gut'
        }
    }
}
