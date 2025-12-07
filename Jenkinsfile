pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                bat """
                python -m pip install --upgrade pip
                pip install flask
                echo "Tests skipped"
                """
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-user',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    script {
                        env.IMAGE_TAG = "${BUILD_NUMBER}"
                    }
                    bat """
                    docker build -t %DOCKER_USER%/todo:%IMAGE_TAG% .
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_USER%/todo:%IMAGE_TAG%
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    bat """
                    kubectl set image deployment/todo-app todo-app=%DOCKER_USER%/todo:%IMAGE_TAG%
                    kubectl rollout status deployment/todo-app
                    """
                }
            }
        }
    }
}
