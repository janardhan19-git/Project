pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        // Nexus Docker registry settings (using HTTP on port 8082)
        DOCKER_REGISTRY = "54.242.130.89:8082"  // Use the Docker connector port (8082)
        DOCKER_REPO = "repository/backend-app"   // Verify this path in Nexus UI
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        // Kubernetes manifests Git repo
        KUBE_MANIFEST_REPO = "https://github.com/tupakulamanoj/kube-manifests.git"
    }
    stages {
        // Stage 1: Build the Java app with Maven
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        // Stage 2: Build Docker image
        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        // Stage 3: Push to Nexus (with HTTP fix)
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        # Logout first to clear old sessions
                        docker logout ${DOCKER_REGISTRY} || true
                        
                        # Login to Nexus (HTTP)
                        echo \$NEXUS_PASS | docker login http://${DOCKER_REGISTRY} -u \$NEXUS_USER --password-stdin
                        
                        # Push the image
                        docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        // Stage 4: Update Kubernetes manifests
        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-cred', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        # Clone manifests repo
                        git clone ${KUBE_MANIFEST_REPO} kube-manifests
                        
                        # Update image tag in deployment.yaml
                        cd kube-manifests
                        sed -i "s|image: .*|image: ${DOCKER_REGISTRY}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml
                        
                        # Commit and push changes
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
