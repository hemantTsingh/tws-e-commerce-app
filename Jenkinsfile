pipeline {
    agent {
        label 'worker-root' }
    
    environment {
        // Update the main app image name to match the deployment file
        DOCKER_IMAGE_NAME = 'hemantsingh1023/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'hemantsingh1023/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github')
        GIT_BRANCH = "master"
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    git url: 'https://github.com/hemantTsingh/tws-e-commerce-app.git',branch: 'master', credentialsId: 'github'
                }
            }
        }
        stage('Build Docker Images') {
    parallel {
        stage('Build Main App Image') {
            steps {
                script {
                    def imageNameWithTag = "${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                    def appImage = docker.build(imageNameWithTag, "-f Dockerfile .")
                }
            }
        }
        // Add other parallel stages like "Build Migration Image" here if needed
    

        //stage('Build Docker Images') {
            //parallel {
                //stage('Build Main App Image') {
                    //steps {
                        //script {
                        //    docker_build(
                       //         imageName: env.DOCKER_IMAGE_NAME,
                      //          imageTag: env.DOCKER_IMAGE_TAG,
                     //           dockerfile: 'Dockerfile',
                    //            context: '.'
                   //         )
                  //      }
                 //   }
                //}
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    // Create directory for results
                  
                    trivy_scan()
                    
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'a43155be-603e-4753-8637-3f9ef04cbef2'
                            )
                        }
                    }
                }
                
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'a43155be-603e-4753-8637-3f9ef04cbef2'
                            )
                        }
                    }
                }
            }
        }
        
        // Add this new stage
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github',
                        gitUserName: 'hemantTsingh',
                        gitUserEmail: 'hemantsingh1023@gmail.com'
                    )
                }
            }
        }
    }
}
