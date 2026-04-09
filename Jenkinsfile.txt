pipeline {
    agent any
    
    triggers {
        githubPush()
    }

    environment {
        SITE_NAME = "jenkinstest"
        WEB_ROOT  = "/var/www/jenkinstest"
        NGINX_CONF = "/etc/nginx/sites-available/jenkinstest"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/kestonbhola/jenkinstest.git/', branch: 'main'
            }
        }

        stage('Verify Project Files') {
            steps {
                sh '''
                    set -e
                    echo "Checking required project files..."

                    test -f index.html
                    test -f sgustyle.css
                    test -f sguscript.js

                    echo "Required files found."
                    ls -la
                '''
            }
        }

        stage('Install Nginx') {
            steps {
                sh '''
                    set -e
                    sudo apt update
                    sudo apt install -y nginx
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

                    rm -rf "$WEB_ROOT"/*
                    cp index.html "$WEB_ROOT"/
                    cp sgustyle.css "$WEB_ROOT"/
                    cp sguscript.js "$WEB_ROOT"/

                    [ -f grenada.jpeg ] && cp grenada.jpeg "$WEB_ROOT"/ || true
                    [ -f grenada-updated.jpeg ] && cp grenada-updated.jpeg "$WEB_ROOT"/ || true

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