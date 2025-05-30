pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_REGISTRY = "3.89.105.109:8082"  // Nexus HTTP port
        DOCKER_REPO = "backend-app"              // Correct repo name
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_MANIFEST_REPO = "https://github.com/janardhan19-git/kube-manifests.git"
    }
    stages {
        stage('Build-Stage') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push into Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        docker logout ${DOCKER_REGISTRY} || true
                        echo \$NEXUS_PASS | docker login http://${DOCKER_REGISTRY} -u \$NEXUS_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Update in Manifest') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        sh """
                            # Check if kube-manifests directory exists, and remove it if it does
                            if [ -d "kube-manifests" ]; then
                                rm -rf kube-manifests
                            fi

                            # Clone the repository using GitHub Token for authentication
                            git clone https://janardhan19-git:${GITHUB_TOKEN}@github.com/janardhan19-git/kube-manifests.git kube-manifests
                            cd kube-manifests
                            sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml
                            git config user.email "janardhanatlassian@gmail.com"
                            git config user.name "janardhan19-git"

                            git add deployment.yaml
                            git commit -m "Update image to ${IMAGE_TAG}"
                            git push origin main
                        """
                    }
                }
            }
        }
    }
}
