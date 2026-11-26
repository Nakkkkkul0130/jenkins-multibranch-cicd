pipeline {
    agent any

    tools {
        nodejs 'NodeJS-7.8.0'
    }

    environment {
        IMAGE_NAME = 'node'
        IMAGE_TAG = 'v1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test -- --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DOCKER_IMAGE = 'nodemain:v1.0'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.DOCKER_IMAGE = 'nodedev:v1.0'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }

                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def appPort = env.BRANCH_NAME == 'main' ? '3000' : '3001'
                    def containerName = env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'

                    sh """
                        docker rm -f ${containerName} 2>/dev/null || true

                        docker run -d \
                            --name ${containerName} \
                            -p ${appPort}:3000 \
                            ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}
