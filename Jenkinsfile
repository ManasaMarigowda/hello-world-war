pipeline {
    agent any

    environment {
        IMAGE_NAME = "manasamarigowda/tomcat"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER_NAME = "tomcat"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                url: 'https://github.com/ManasaMarigowda/hello-world-war.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-key') {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker pull ${IMAGE_NAME}:latest

                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d \
                --name ${CONTAINER_NAME} \
                -p 9000:8080 \
                ${IMAGE_NAME}:latest
                """
            }
        }
    }
}
