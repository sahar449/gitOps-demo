pipeline{
    agent any
    environment{
        DOCKERHUB_USERNAME = "sahar449"
        DOCKERHUB_LOGIN = "docker_hub_login"
        APP_NAME = "gitops-demo-app"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    }
    stages{
        stage('Build docker and tag'){
            steps{
                script{
                    sh """
                        docker build -t ${APP_NAME}:$BUILD_ID .
                        docker image tag ${APP_NAME}:$BUILD_ID ${IMAGE_NAME}:$BUILD_ID
                        docker image tag ${APP_NAME}:$BUILD_ID ${IMAGE_NAME}:latest
                        """
                }
            }
        }
        stage('docker login and push'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_hub_login', variable: 'docker_hub')]) {
                        sh """
                                docker login -u sahar449 -p ${docker_hub}
                                docker push ${IMAGE_NAME}:$BUILD_ID
                                docker push ${IMAGE_NAME}:latest
                            """
                    }
                }
            }
        }
        stage('Installtion by helm'){
            steps{
                script{
                    sh """
                        helm create ci 
                        cd ci
                        helm install ci .
                        echo $(curl minikube.com:30011/health)
                        """
                }
            }
        }
        stage('update k8s deployment'){
            steps{
                script{
                    sh """
                        cat deployment.yml    
                        sed -i 's/${APP_NAME}.*/${APP_NAME}:${BUILD_ID}/g' deployment.yml
                        cat deployment.yml
                     """
                }
            }
        }
        stage('push the update to github'){
            steps{
                script{
                    withCredentials([gitUsernamePassword(credentialsId: 'github_pat', gitToolName: 'Default')]) {
                            sh """
                                git add .
                                git commit -m "update deployment"
                                git push origin HEAD:main
                                """
                        }
                    }
                }
            }
        }    
    }
