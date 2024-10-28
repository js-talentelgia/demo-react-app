pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('jagseersingh')
        GITHUB_CREDENTIALS_ID = 'js-talentelgia'
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("jagseersingh/react-app:${env.BUILD_ID}")
                }
            }
        }
        stage('Login to Docker Hub') {         
            steps {                            
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                 
                echo 'Login Completed'                
            }           
        }
        stage('Push') {
            steps {                            
                sh "docker push jagseersingh/react-app:${env.BUILD_ID}"
                echo 'Push Image Completed'       
            }    
        }
        stage('Update Manifests') {
            steps {
                script {
                    sh """
                    sed -i 's|image: jagseersingh/react-app:.*|image: jagseersingh/react-app:${env.BUILD_ID}|' k8s/deployment.yaml
                    """
                }
            }
        }
        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'js-talentelgia', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                        git config user.email "jenkins@example.com" || exit 1
                        git config user.name "Jenkins" || exit 1
                        git add k8s/deployment.yaml || exit 1
                        git commit -m "Update image tag to ${env.BUILD_ID}" || exit 1
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/js-talentelgia/demo-react-app.git HEAD:main || exit 1
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend color: 'good', message: "Build SUCCESS: ${currentBuild.fullDisplayName} [${env.BUILD_NUMBER}] (<${env.BUILD_URL}|Open>)"
        }
        failure {
            slackSend color: 'danger', message: "Build FAILURE: ${currentBuild.fullDisplayName} [${env.BUILD_NUMBER}] (<${env.BUILD_URL}|Open>)"
        }
        always {
            sh 'docker logout || exit 1'
            sh "docker rmi jagseersingh/react-app:${env.BUILD_ID} || true" // Remove the Docker image after the build
            deleteDir() // Clear the workspace
        }
    }
}
