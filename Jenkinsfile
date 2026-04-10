pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SITE_NAME  = "jenkinstest"
        WEB_ROOT   = "/var/www/jenkinstest"
        NGINX_CONF = "/etc/nginx/sites-available/jenkinstest"
        REPO_URL   = "https://github.com/AceKid13/iss-cybersecurity-resources.git"
        BRANCH_NAME = "main"
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

                    echo "Required files found."
                    echo "Project contents:"
                    ls -la
                '''
            }
        }

        stage('Install Nginx') {
            steps {
                sh '''
                    set -e
                    sudo apt update
                    sudo apt install -y nginx rsync
                '''
            }
        }

        stage('Create Web Root') {
            steps {
                sh '''
                    set -e
                    sudo mkdir -p "$WEB_ROOT"
                    sudo chown -R jenkins:jenkins "$WEB_ROOT"
                '''
            }
        }

        stage('Deploy Website Files') {
            steps {
                sh '''
                    set -e

                    echo "Cleaning old deployed files..."
                    rm -rf "$WEB_ROOT"/*

                    echo "Copying website files..."
                    rsync -av --delete \
                        --exclude='.git' \
                        --exclude='Jenkinsfile' \
                        --exclude='Jenkinsfile.txt' \
                        ./ "$WEB_ROOT"/

                    echo "Deployed files:"
                    ls -la "$WEB_ROOT"
                '''
            }
        }

        stage('Configure Nginx Site') {
            steps {
                sh '''
                    set -e

                    sudo bash -c "cat > $NGINX_CONF" <<EOF
server {
    listen 80;
    server_name _;

    root $WEB_ROOT;
    index index.html;

    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF

                    sudo ln -sf "$NGINX_CONF" /etc/nginx/sites-enabled/jenkinstest
                    sudo rm -f /etc/nginx/sites-enabled/default

                    sudo nginx -t
                '''
            }
        }

        stage('Start Nginx') {
            steps {
                sh '''
                    set -e
                    sudo systemctl enable nginx
                    sudo systemctl restart nginx
                    sudo systemctl status nginx --no-pager
                '''
            }
        }

        stage('Test Website Locally') {
            steps {
                sh '''
                    set -e
                    curl -I http://localhost
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful.'
            echo 'Open your EC2 public IP in a browser to view the site.'
        }
        failure {
            echo 'Deployment failed. Check the Jenkins console output.'
        }
    }
}
