pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/tuckert4'                                                 // <------change this
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/tuckert4-oss/225-lab3-2.git'                   // <------change this
        KUBECONFIG = credentials('tuckert4')                                             // <------change this
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment using Ingress') {
            steps {
                script {
                    // Update deployment-dev.yaml to use the new image tag
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh "kubectl apply -f deployment-dev.yaml"
                }
            }
        }
 
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get all"
                }
            }
        }
    }
    post {
        success {
            slackSend([color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
        }
        unstable {
            slackSend([color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
        }
        failure {
            slackSend([color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
    }
}
}
