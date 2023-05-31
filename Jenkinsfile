pipeline{
    agent any
    environment{
        DOCKERHUB_USERNAME = "sahar449"
        DOCKERHUB_LOGIN = "docker_hub_login"
        APP_NAME = "gitops-demo-app"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    }
    stages{
        stage('Docker login, build, tag and push'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_hub_login', variable: 'docker_hub')]) {
                    sh """
                        docker build -t ${APP_NAME}:$BUILD_ID .
                        docker image tag ${APP_NAME}:$BUILD_ID ${IMAGE_NAME}:$BUILD_ID
                        docker image tag ${APP_NAME}:$BUILD_ID ${IMAGE_NAME}:latest
                        docker login -u sahar449 -p ${docker_hub}
                        docker push ${IMAGE_NAME}:$BUILD_ID
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
        stage('Deploy to argocd and check helathly'){
            steps{
                script{
                    withKubeConfig([credentialsId: 'kubeconfig']){
                        sh '''  kubectl apply -f argocd.yml
                                echo $(curl http://minikube.com:32000/health)
                            '''

                    }
                }
            }
        }
    }
    post {
		    always {
                    withCredentials([string(credentialsId: 'my_email', variable: 'my_email')]) {
		 	            mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "${my_email}";  
                }
            }
	    }
}
