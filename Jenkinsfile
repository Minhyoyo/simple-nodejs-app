pipeline {
    agent any
    
    environment {
        HARBOR_URL = "fine-quail-balanced.ngrok-free.app"
        HARBOR_CREDENTIAL_ID = "harbor-credentials"
        IMAGE_NAME = "minhyoyo/simple-nodejs-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${HARBOR_CREDENTIAL_ID}", passwordVariable: 'HARBOR_PASSWORD', usernameVariable: 'HARBOR_USERNAME')]) {
                    sh "echo ${HARBOR_PASSWORD} | docker login ${HARBOR_URL} -u ${HARBOR_USERNAME} --password-stdin"
                    sh "docker push ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Clone the GitOps repository
                    sh "git clone https://github.com/yourusername/gitops-repo.git"
                    
                    // Update image tag in Kubernetes manifests
                    sh "sed -i 's|image: ${HARBOR_URL}/${IMAGE_NAME}:.*|image: ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG}|' gitops-repo/apps/nodejs-app/deployment.yaml"
                    
                    // Commit and push changes
                    dir('gitops-repo') {
                        sh "git config user.email 'jenkins@example.com'"
                        sh "git config user.name 'Jenkins'"
                        sh "git add ."
                        sh "git commit -m 'Update image to ${IMAGE_TAG}'"
                        withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yourusername/gitops-repo.git HEAD:main"
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker logout ${HARBOR_URL}"
            cleanWs()
        }
    }
}
