pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'vikasavaws/nginx-jenkins-kind'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vikasavaws/nginx-jenkins-kind.git'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    test -f Dockerfile
                    test -f index.html
                    test -f k8s/deployment.yaml
                    test -f k8s/service.yaml
                    echo "All required files exist"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/nginx \
                      nginx=${DOCKER_IMAGE}:${BUILD_NUMBER}

                    kubectl rollout status deployment/nginx \
                      --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get deployment nginx
                    kubectl get service nginx
                '''
            }
        }
    }
}
