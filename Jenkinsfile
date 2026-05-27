pipeline {

    agent any

    environment {
        IMAGE_NAME = geetaswapna/myapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Source') {
            steps {
                git 'https://github.com/geetaswapna/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
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
                sh 'docker push $IMAGE_NAME:$TAG'
            }
        }

        stage('Update Manifest Repo') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh '''

                    git clone https://$GIT_USER:$GIT_PASS@github.com/username/k8s-manifests.git

                    sed -i "s|image:.*|image: $IMAGE_NAME:$TAG|g" \
                    k8s-manifests/deployment.yaml

                    cd k8s-manifests

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
