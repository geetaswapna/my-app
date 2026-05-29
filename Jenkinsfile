pipeline {

    agent any

    environment {
        IMAGE_NAME = "geetaswapna/my-nginx"
        TAG = "v2"
    }

    stages {

        stage('Clone Source') {
            steps {
                git branch: 'main',
                url: 'https://github.com/geetaswapna/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t geetaswapna/my-nginx:v3 .'
            }
        }

        stage('Docker Login') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login \
                    -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push geetaswapna/my-nginx:v3'
            }
        }

        stage('Update Deployment File') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh """

                    rm -rf k8-auto

                    git clone https://\$GIT_USER:\$GIT_PASS@github.com/geetaswapna/k8-auto.git

                    sed -i 's|image:.*|image: geetaswapna/my-nginx:v3|g' k8-auto/deployment.yml

                    cat k8-auto/deployment.yml

                    cd k8-auto

                    git config user.email "jenkins@gmail.com"
                    git config user.name "jenkins"

                    git add .

                    git commit -m "Updated image to v2" || true

                    git push origin main

                    """
                }
            }
        }
    }
}

