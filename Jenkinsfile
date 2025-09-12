pipeline {
    agent {
        label 'worker'
    }

    environment {
        DOCKER_IMAGE_NAME = 'hemantsingh1023/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'hemantsingh1023/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_BRANCH = "master"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/hemantTsingh/tws-e-commerce-app.git',
                    branch: "${env.GIT_BRANCH}",
                    credentialsId: 'github'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f Dockerfile ."
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        sh "docker build -t ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f scripts/Dockerfile.migration ."
                    }
                }
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh "trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"
                sh "trivy image ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            '''
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            '''
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config --global user.name 'hemantTsingh'
                        git config --global user.email 'hemantsingh1023@gmail.com'

                        sed -i 's|image: .*easyshop-app:.*|image: hemantsingh1023/easyshop-app:'"$DOCKER_IMAGE_TAG"'|' kubernetes/08-easyshop-deployment.yaml
                        

                        git add kubernetes/08-easyshop-deployment.yaml 
                        git diff --cached --quiet || git commit -m "Update image tags to ${DOCKER_IMAGE_TAG}"
                        git push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/hemantTsingh/tws-e-commerce-app.git HEAD:${GIT_BRANCH}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    withEnv(["KUBECONFIG=$KUBECONFIG_FILE"]) {
                        sh '''
                            for file in $(ls kubernetes/*.yaml); do
    kubectl apply -f "$file"
done
                        '''
                    }
                }
            }
        }
    }
}
