pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_REGISTRY = "54.242.130.89:8082"  // Correct - Nexus HTTP port
        DOCKER_REPO = "backend-app"              // Correct repo name
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_MANIFEST_REPO = "https://github.com/tupakulamanoj/kube-manifests.git"
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }
        stage('Docker Push') {
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
        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([string(credentialsId: 'github-creds', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Ensure the repository directory is empty before cloning
                        sh """
                            if [ -d "kube-manifests" ]; then
                                rm -rf kube-manifests
                            fi

                            // Clone the repository using GitHub Token for authentication
                            git clone https://tupakulamanoj:${GITHUB_TOKEN}@github.com/tupakulamanoj/kube-manifests.git kube-manifests
                            cd kube-manifests
                            sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml
                            git config user.email "manojthupakula06080@gmail.com"
                            git config user.name "tupakulamanoj"

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
