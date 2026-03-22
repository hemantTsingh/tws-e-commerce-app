pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'hemantsingh1023/easyshop-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USER = 'hemantsingh1023'
        DOCKERHUB_PASS = 'dckr_pat_MrWFMAN9gb0oPLrsJ2jdvL9uy2w'
    }

    stages {
        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Clone') {
            steps {
                git url: 'https://github.com/hemantTsingh/tws-e-commerce-app.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push') {
            steps {
                sh '''
                    echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Update Manifest') {
            steps {
                sh "sed -i 's|image: hemantsingh1023/easyshop-app:.*|image: hemantsingh1023/easyshop-app:${DOCKER_TAG}|' kubernetes/08-easyshop-deployment.yaml"
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f kubernetes/'
            }
        }
    }
}
