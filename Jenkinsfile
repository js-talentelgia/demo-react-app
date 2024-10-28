pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS= credentials('jagseersingh')
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
            steps{                            
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                 
                echo 'Login Completed'                
            }           
        }
        stage('Push') {
            steps{                            
                sh "docker push jagseersingh/react-app:${env.BUILD_ID}"
                echo 'Push Image Completed'       
            }    
        }
        stage('Update Manifests') {
            steps {
                script {
                    sh """
                    sed -i 's|image: jagseersingh/react-app:.*|image: jagseersingh/react-app:${env.BUILD_ID}|' deployment.yaml
                    """
                }
            }
        }
        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'js-talentelgia', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image tag to ${env.BUILD_ID}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/js-talentelgia/demo-react-app.git HEAD:main
                        """
                    }
                }
            }
        }
    }
}
