pipeline {
    agent any

     environment {
        IMAGE_NAME = "manasamarigowda/tomcat"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER_NAME = "tomcat-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/ManasaMarigowda/hello-world-war.git'
            }
        }

 
        stage('Install Docker') {
            steps {
                sh '''
                    # Install docker only if not installed
                    if ! command -v docker >/dev/null 2>&1; then
                        echo "Installing Docker..."
                        sudo apt update
                        sudo apt install docker.io -y
                        sudo systemctl start docker
                        sudo systemctl enable docker
                        sudo usermod -aG docker jenkins
                    else
                        echo "Docker already installed"
                    fi
                '''
            }
        }

        
        stage('Build') {
            steps {
                script {
                    app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    docker.withRegistry('', 'docker-key') {
                        app.push("${IMAGE_TAG}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker pull ${IMAGE_NAME}:latest

                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 8080:8080 \
                      ${IMAGE_NAME}:latest
                """
            }
        }
    }
}
