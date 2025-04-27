pipeline {
    agent any

    tools {
        maven 'Maven'  // Ensure you have configured Maven in Jenkins tools
    }

    environment {
        DOCKER_REGISTRY = "54.242.130.89:8082"  // Correct Nexus HTTP port
        DOCKER_REPO = "backend-app"             // Correct repo name
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBE_MANIFEST_REPO = "https://github.com/tupakulamanoj/kube-manifests.git"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'  // Clean and package using Maven
            }
        }

        stage('Docker Build') {
            steps {
                // Building Docker image from Dockerfile
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    // Docker login to Nexus registry
                    sh """
                        docker logout ${DOCKER_REGISTRY} || true
                        echo \$NEXUS_PASS | docker login ${DOCKER_REGISTRY} -u \$NEXUS_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-cred', keyFileVariable: 'SSH_KEY')]) {
                    // Clone Kubernetes manifest repository, update the image, and push changes
                    sh """
                        git clone ${KUBE_MANIFEST_REPO} kube-manifests
                        cd kube-manifests
                        sed -i "s|image: .*|image: ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml
                        git config user.email "manojthupakula06080@gmail.com"
                        git config user.name "tupakulamanoj"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_TAG}"
                        GIT_SSH_COMMAND="ssh -i ${SSH_KEY}" git push origin main
                    """
                }
            }
        }
    }
}
