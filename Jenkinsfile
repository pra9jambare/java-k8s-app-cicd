pipeline {
    agent any

    environment {
        IMAGE_NAME = "pranavjambare/java-k8s-app"
        TAG = "${BUILD_NUMBER}"
        IMAGE = "${IMAGE_NAME}:${TAG}"
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
    }

    tools {
        maven "Maven-3"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE} .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                    chmod +x scripts/trivy-scan.sh
                    ./scripts/trivy-scan.sh
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: '28658194-41d0-467e-b36f-b7563a12baff',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh """
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/java-k8s-app \
                        java-k8s-app=${IMAGE}

                    kubectl rollout status deployment/java-k8s-app --timeout=120s
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    kubectl get pods
                    kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS: ${IMAGE} deployed"
        }

        failure {
            echo "Pipeline FAILED"
        }

        always {
            cleanWs()
        }
    }
}
