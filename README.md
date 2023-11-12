# Mini DevOps Project
## Overview

This mini-project demonstrates a full DevOps pipeline, including Docker containerization, Jenkins CI/CD, basic unit testing, and Kubernetes deployment for a web application with separate backend and frontend components.
![Project Architecture](images/arch.gif)

## Dockerization

### Backend Dockerfile

```shell
    # Starting from a node base image
    FROM node:18-alpine

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json
    COPY package*.json ./

    # Install dependencies
    RUN npm install


    # Copy the rest of the application code
    COPY . .

    # Expose the port the app runs on
    EXPOSE 3001

    # Command to run the app
    CMD [ "node", "index.js" ]

```
![Backend Dockerfile Build](images/backbuildandpush.PNG)
### Frontend Dockerfile

```shell
    # Starting from a node base image
    FROM node:18-alpine as build

    # Set the working directory in the container
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json
    COPY package*.json ./

    # Install dependencies
    RUN npm install

    # Copy the rest of the frontend code
    COPY . .

    # Build the application
    RUN npm run build

    # Start a new stage from nginx to serve the static files
    FROM nginx:alpine

    # Copy static assets from builder stage
    COPY --from=build /usr/src/app/build /usr/share/nginx/html

    # Expose port 80 to access the container
    EXPOSE 80

    # No need for CMD because it's included in the nginx base image


```
![Backend Dockerfile Build](images/frontbuild.PNG)
![Backend Dockerfile Build](images/frontpush.PNG)


## Jenkins Pipeline

The Jenkins Pipeline automates the process of building Docker images for both the backend and frontend, running unit tests, and pushing the images to a Docker registry. The pipeline is defined in a `Jenkinsfile`, which contains multiple stages to handle different parts of the process.

### Pipeline Stages

1. **Checkout**: The pipeline checks out the code from the GitHub repository.

2. **Build Backend**: This stage builds the Docker image for the backend from the Dockerfile located in the `/server` directory.

    ```groovy
   stage('Build Backend') {
        steps {
            script {
                dir('server') {
                    sh 'ls -la' // Lists all files in the server directory
                    sh 'docker build -t boudhraadhia7/myapp-backend .'
                }
            }
        }
    }
    ```

3. **Build Frontend**: Similar to the backend, this stage builds the Docker image for the frontend from the Dockerfile in the `/client` directory.

    ```groovy
      stage('Build Frontend') {
        steps {
            script {
                dir('client') {
                    sh 'ls -la' // Lists all files in the server directory
                    sh 'docker build -t boudhraadhia7/myapp-frontend .'
                }
            }
        }
    }
    ```

4. **Unit Tests**: Runs unit tests for both backend and frontend. The tests are located in their respective directories.

    ```groovy
    stage('Unit Tests') {
        steps {
            script {
                dir('client') {
                   sh 'ls -la' // Lists all files in the server directory
                   sh 'npm test'
                }
            }
            script {
                dir('server') {
                   sh 'npm install'
                   sh 'npm test'
                }
            }
            sh 'echo "Unit tests passed"'
        }
    }
    ```

5. **Push to Registry**: This stage logs into Docker Hub and pushes both backend and frontend images to the Docker registry.

    ```groovy
    stage('Push to Registry') {
        steps {
            script {
                docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                    docker.image("myapp-backend:latest").push()
                    docker.image("myapp-frontend:latest").push()
                }
            }
        }
    }
    ```

6. **Deploy**: In this stage, the application is deployed to a Kubernetes cluster using kubectl commands.

    ```groovy
    stage('Deploy') {
        steps {
            sh 'kubectl apply -f k8s-manifest.yaml'
        }
    }
    ```

### Jenkinsfile

The `Jenkinsfile` is located at the root of the repository and is configured as a Declarative Pipeline in Jenkins.

![Jenkins Pipeline](path/to/jenkins_pipeline_screenshot.png)

### Pipeline Execution

The pipeline is triggered on each commit to the repository, ensuring that every change is automatically built, tested, and ready for deployment.

![Pipeline Execution](path/to/pipeline_execution.gif)



### Deployment Process

Explain the steps for deploying the application on Kubernetes, including setting up the environment, applying Kubernetes manifests, and verifying the deployment.

```bash
# Kubernetes deployment commands
kubectl apply -f k8s-manifest.yaml
