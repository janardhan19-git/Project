pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        // Update this line to use port 8082 instead of 8081
        DOCKER_REGISTRY = "54.242.130.89:8082"
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
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        echo \$NEXUS_PASS | docker login ${DOCKER_REGISTRY} -u \$NEXUS_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-cred', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        git clone ${KUBE_MANIFEST_REPO} kube-manifests
                        cd kube-manifests
                        sed -i "s|image: .*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml
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
