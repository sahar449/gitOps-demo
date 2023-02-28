pipeline{
    agent any
    environment{
        DOCKERHUB_USERNAME = "sahar449"
        DOCKERHUB_LOGIN = "docker_hub_login"
        APP_NAME = "gitops-demo-app"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    }
    stages{
        // stage('Clean workspace'){
        //     steps{
        //         script{
        //             cleanWs()
        //         }
        //     }
        // }
        
        stage('stop all the containers'){
            steps{
                script{
                    sh "docker container prune --force"
                }
            }
        }
        stage('Build docker and tag'){
            steps{
                script{
                    sh """
                        docker build -t ${APP_NAME}:v1.$BUILD_ID .
                        docker image tag ${APP_NAME}:v1.$BUILD_ID ${IMAGE_NAME}:v1.$BUILD_ID
                        docker image tag ${APP_NAME}:v1.$BUILD_ID ${IMAGE_NAME}:latest
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
                                docker push ${IMAGE_NAME}:v1.$BUILD_ID
                                docker push ${IMAGE_NAME}:latest
                            """
                    }
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
                    sh """
                    git config --global user.name "sahar449"
                    git config --global user.email "saharr449@gmail.com"
                    git add deployment.yml
                    git commit -m 'update deployment' """
                    withCredentials([gitUsernamePassword(credentialsId: 'github_pat', gitToolName: 'Default')]) {
                            sh "git push git@github.com:sahar449/gitOps-demo.git"
                        }
                    }
                }
            }
        }
    }
