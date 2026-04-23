pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        REPO_URL       = "https://github.com/AceKid13/iss-cybersecurity-resources.git"
        BRANCH_NAME    = "main"
        IMAGE_NAME     = "jenkinstest-image"
        CONTAINER_NAME = "jenkinstest-container"
        HOST_PORT      = "8081"
        CONTAINER_PORT = "80"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git url: "${REPO_URL}", branch: "${BRANCH_NAME}"
            }
        }

        stage('Verify Project Files') {
            steps {
                sh '''
                    set -e
                    echo "Checking required project files..."

                    test -f index.html
                    test -f style.css
                    test -f Dockerfile
                    test -f nginx.conf

                    echo "Required files found."
                    echo "Project contents:"
                    ls -la
                '''
            }
        }

        stage('Check Docker') {
            steps {
                sh '''
                    set -e
                    docker --version
                    docker ps
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    docker build -t "$IMAGE_NAME" .
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    set +e
                    docker stop "$CONTAINER_NAME"
                    docker rm "$CONTAINER_NAME"
                    set -e
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                    set -e
                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      -p "$HOST_PORT":"$CONTAINER_PORT" \
                      --restart unless-stopped \
                      "$IMAGE_NAME"
                '''
            }
        }

        stage('Test Website Locally') {
            steps {
                sh '''
                    set -e
                    curl -I http://localhost:8081
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker deployment successful.'
            echo 'Open your EC2 public IP in a browser with port 8081.'
        }
        failure {
            echo 'Deployment failed. Check the Jenkins console output.'
        }
    }
}
