
pipeline {
    agent any

    environment {
        IMAGE = 'sanjeevsekar23/docker2'
        DOCKER = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\Admin\\.kube\\config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Sanjeevsekar23/docker2.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                }
                bat '"%DOCKER%" build -t %IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    bat 'echo %DOCKER_PASSWORD%| "%DOCKER%" login -u %DOCKER_USERNAME% --password-stdin'
                    bat '"%DOCKER%" push %IMAGE%:%IMAGE_TAG%'
                }
            }
        }

        stage('Deploy') {
            steps {
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" set image deployment/department-portal department-portal=%IMAGE%:%IMAGE_TAG%'
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" rollout status deployment/department-portal --timeout=180s'
            }
        }

        stage('Verify') {
            steps {
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" get deployments,pods,services'
            }
        }
    }

    post {
        always {
            bat '"%DOCKER%" logout'
        }
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
