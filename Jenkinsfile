pipeline {
    agent {
        label 'worker-root'
    }

    environment {
        DOCKER_IMAGE_NAME = 'hemantsingh1023/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'hemantsingh1023/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github')
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
                        script {
                            sh """
                                docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f Dockerfile .
                            """
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            sh """
                                docker build -t ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f scripts/Dockerfile.migration .
                            """
                        }
                    }
                }
            }
        }

   ///     stage('Run Unit Tests') {
      //      steps {
        //        script {
          //          // Replace with your actual test script/command
             //       sh './scripts/run_tests.sh'
            //    }
            //}
       // }
///
        ///stage('Security Scan with Trivy') {
            ///steps {
                //script {
                    // Replace with actual Trivy scan command
               //     sh "trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"
             //       sh "trivy image ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"
           //     }
          //  }
        //}

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'a43155be-603e-4753-8637-3f9ef04cbef2', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh """
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            """
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'a43155be-603e-4753-8637-3f9ef04cbef2', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh """
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            """
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Replace with your actual implementation or shell script
                    sh """
                        git config --global user.name 'hemantTsingh'
                        git config --global user.email 'hemantsingh1023@gmail.com'

                        cd kubernetes

                        sed -i 's|image: hemantsingh1023/easyshop-app:.*|image: hemantsingh1023/easyshop-app:${DOCKER_IMAGE_TAG}|' 08-easyshop-deployment.yaml
                        sed -i 's|image: hemantsingh1023/easyshop-migration:.*|image: hemantsingh1023/easyshop-migration:${DOCKER_IMAGE_TAG}|' 12-migration-job.yaml

                        git add .
                        git diff --cached --quiet || git commit -m "Update image tags to ${DOCKER_IMAGE_TAG}"
                        git push https://$GITHUB_CREDENTIALS_USR:$GITHUB_CREDENTIALS_PSW@github.com/hemantTsingh/tws-e-commerce-app.git HEAD:${GIT_BRANCH}
                    """
                }
            }
        }
    }
}
