pipeline {
    agent any

    environment {
        DOCKER_USER = 'rithwik57'
        IMAGE_NAME  = 'flask-web-app'
        IMAGE_TAG   = "${env.BUILD_ID}"
        REGISTRY    = "docker.io/${DOCKER_USER}/${IMAGE_NAME}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rithwik57/flask-jenkins-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image and tag with BUILD_ID
                    app = docker.build(
                        "${REGISTRY}:${IMAGE_TAG}",
                        "--build-arg BUILD_ID=${env.BUILD_ID} ."
                    )
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove old container if exists
                    sh "docker stop ${IMAGE_NAME} || true"
                    sh "docker rm ${IMAGE_NAME} || true"

                    // Run new container
                    sh """
                    docker run -d \
                        --name ${IMAGE_NAME} \
                        -p 5000:5000 \
                        -e BUILD_ID=${env.BUILD_ID} \
                        ${REGISTRY}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up image after build
            sh "docker rmi ${REGISTRY}:${IMAGE_TAG} || true"
        }
    }
}
