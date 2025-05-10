pipeline {
    agent any

    enviroment {
        DOCKER_IMAGE_NAME = 'python_webapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
        CONTAINER_NAME = 'python_webapp_container'
    }

    options {
        timestamps() 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Building Docker Image') {
            steps {
                dir('pythonm_webapp') {
                    script {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }
        stage('Testing') {
            steps {
                script {
                    sh "docker run --name test_${CONTAINER_NAME} -d -p 5001:5000 ${DCOKER_IMAGE}"
                    sh "sleep 10"
                    sh "curl -s http://localhost:5001/"
                    sh "docker stop test_${CONTAINER_NAME} || true"
                    sh "docker rm test_${CONTAINER_NAME} || true"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}