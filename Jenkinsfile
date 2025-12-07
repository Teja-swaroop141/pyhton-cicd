pipeline {
    agent any

    environment {
        DOCKER_USER = credentials('dockerhub-user')
        DOCKER_PASS = credentials('dockerhub-pass')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Teja-swaroop141/pyhton-cicd.git'
            }
        }

        stage('Test') {
            steps {
                sh 'pip install flask pytest'
                sh 'pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_USER}/todo:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                sh "docker push ${DOCKER_USER}/todo:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment.yml'
                }
            }
        }
    }
}
