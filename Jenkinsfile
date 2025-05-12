pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'python_webapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "balcus/${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
        CONTAINER_NAME = 'python_webapp_container'
        DOCKERHUB_CREDENTIALS = credentials('balcus-dockerhub')
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

        stage('Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Linting') {
            steps {
                script {
                    echo 'Running linting'
                    sh '''
                    pip install pylint --break-system-packages
                    export PATH=$HOME/.local/bin:$PATH
                    pylint --exit-zero app.py
                    '''
                }
            }
        }

        stage('Testing') {
            steps {
                script {
                    sh "docker run --name test_${CONTAINER_NAME} -d -p 5001:5000 ${DOCKER_IMAGE}"
                    sh "sleep 10"
                    sh "curl -s http://localhost:5001/"
                    sh "docker stop test_${CONTAINER_NAME} || true"
                    sh "docker rm test_${CONTAINER_NAME} || true"
                }
            }
        }

        stage('Login') {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            script {
                sh 'docker logout'
                sh "docker rmi ${DOCKER_IMAGE} || true"
            }
            cleanWs()
        }
    }
}