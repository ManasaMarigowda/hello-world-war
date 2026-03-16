pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/dockerhub-username"
        IMAGE_NAME = "hello-world"
        TAG = "${BUILD_NUMBER}"
        HELM_CHART = "hello-chart"
        JFROG_REPO = "https://jfrog.example.com/artifactory/helm"
        RELEASE_NAME = "hello-release"
        NAMESPACE = "default"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ManasaMarigowda/hello-world-app.git'
            }
        }

        stage('Build Docker Image & Helm Package') {
            steps {
                script {

                    sh """
                    docker build -t $REGISTRY/$IMAGE_NAME:$TAG app/
                    """

                    sh """
                    helm package helm/$HELM_CHART
                    """
                }
            }
        }

        stage('Push Image to Registry & Chart to JFrog') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'docker-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS')]) {

                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:$TAG
                    """
                }

                sh """
                curl -u user:password \
                -T $HELM_CHART-*.tgz \
                $JFROG_REPO/
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                sh """
                helm repo add jfrog $JFROG_REPO
                helm repo update
                """

                sh """
                helm upgrade --install $RELEASE_NAME jfrog/$HELM_CHART \
                --namespace $NAMESPACE \
                --set image.repository=$REGISTRY/$IMAGE_NAME \
                --set image.tag=$TAG
                """
            }
        }
    }
}
