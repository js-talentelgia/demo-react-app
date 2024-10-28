pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'jagseersingh'
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
                    dockerImage = docker.build("jagseersingh/react-repo:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_CREDENTIALS_ID') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Manifests') {
            steps {
                script {
                    sh """
                    sed -i 's|image: jagseersingh/react-repo:.*|image: jagseersingh/react-repo:${env.BUILD_ID}|' deployment.yaml
                    """
                }
            }
        }
        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'GITHUB_CREDENTIALS_ID', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
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
