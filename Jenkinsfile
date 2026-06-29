pipeline {
    agent any

    environment {
        IMAGE_NAME = "pranavjambare/java-k8s-app"
        TAG = "${BUILD_NUMBER}"
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
                sh '''
                    docker build -t ${IMAGE_NAME}:${TAG} .
                '''
            }
        }
        stage('Trivy Scan') {
            steps {
                sh '''
                    chmod +x scripts/trivy-scan.sh
                    ./scripts/trivy-scan.sh
                '''
            }
        }


        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'DockerHub Creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE_NAME}:${TAG}
                        docker logout
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}
