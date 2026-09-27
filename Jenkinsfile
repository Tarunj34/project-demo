pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "tarun4642/technova-website"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Tarunj34/project-demo.git'
            }
        }

        stage('Check Website Files') {
            steps {
                sh '''
                    set -e
                    ls -lh
                    test -s index.html
                    test -s style.css
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    docker build --no-cache \
                        -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -e
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    set -e
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    set -e

                    docker rm -f cont1 || true

                    docker run -d \
                        --name cont1 \
                        -p 7777:80 \
                        ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Verify Container') {
            steps {
                sh '''
                    set -e
                    docker ps
                    docker exec cont1 ls -lh /usr/share/nginx/html/
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            echo "Website: http://EC2_PUBLIC_IP:7777"
            echo "Docker Image: ${DOCKER_IMAGE}:latest"
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
