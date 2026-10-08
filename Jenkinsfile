pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sanjeevsekar23/docker2'
        KUBECONFIG = credentials('kubeconfig')
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Optional Docker image tag; defaults to the Jenkins build number.')
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Sanjeevsekar23/docker2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = params.IMAGE_TAG ?: env.BUILD_NUMBER
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
                    env.IMAGE_TAG = tag
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl rollout status deployment/department-portal --timeout=180s'
                sh 'kubectl get pods,svc -l app=department-portal'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo 'Department portal successfully built, pushed, and deployed.'
        }
        failure {
            echo 'The Jenkins pipeline failed. Check the build logs for details.'
        }
    }
}
