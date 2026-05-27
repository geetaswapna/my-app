pipeline {

    agent any

    environment {
        IMAGE_NAME = "geetaswapna/myapp"
        TAG = "${BUILD_NUMBER}"
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
                sh 'docker build -t geetaswapna/myapp:v1 .'
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
                sh 'docker push geetaswapna/myapp:v1'
            }
        }

        stage('Update Manifest Repo') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'github',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh '''

                    rm -rf k8-auto

                    git clone https://$GIT_USER:$GIT_PASS@github.com/geetaswapna/k8-auto.git

                    sed -i "s|image:.*|image: geetaswapna/myapp:v1|g" \
                    k8-auto/deployment.yml

                    cd k8-auto

                    git config user.email "jenkins@gmail.com"
                    git config user.name "jenkins"

                    git add .

                    git commit -m "Updated image"

                    git push
                    '''
                }
            }
        }
    }
}
