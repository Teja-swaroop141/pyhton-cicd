pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Uses the same repo this Jenkinsfile came from
                checkout scm
            }
        }

        stage('Test') {
            steps {
                bat """
                python -m pip install --upgrade pip
                pip install flask pytest
                pytest
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
                    bat """
                    docker build -t %DOCKER_USER%/todo:latest .
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_USER%/todo:latest
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
                    kubectl apply -f deployment.yml
                    kubectl rollout status deployment/todo-app
                    """
                }
            }
        }
    }
}
