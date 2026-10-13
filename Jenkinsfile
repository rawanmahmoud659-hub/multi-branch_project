pipeline {
    agent any

    environment {
        APP_BASE_NAME = 'app-hello-nti'
    }

    stages {
        stage('Configure Branch') {
            steps {
                script {
                    env.GIT_BRANCH_NAME = env.BRANCH_NAME

                    if (env.GIT_BRANCH_NAME == 'main') {
                        env.APP_NAME = env.APP_BASE_NAME
                    } else {
                        env.APP_NAME =
                            "${env.APP_BASE_NAME}-${env.GIT_BRANCH_NAME}"
                    }

                    env.IMAGE_NAME =
                        "${env.APP_NAME}:${env.BUILD_NUMBER}"

                    env.CONTAINER_NAME =
                        "${env.APP_NAME}-${env.BUILD_NUMBER}"

                    echo "Branch: ${env.GIT_BRANCH_NAME}"
                    echo "App: ${env.APP_NAME}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t "$IMAGE_NAME" .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -eu

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" --password-stdin

                        docker tag "$IMAGE_NAME" \
                            "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"

                        docker push \
                            "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"

                        docker logout
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh '''
                        docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                        docker run -d \
                            -p 5000 \
                            --name "$CONTAINER_NAME" \
                            "$IMAGE_NAME"

                        echo "Container started:"
                        docker ps --filter "name=$CONTAINER_NAME"

                        echo "Assigned port:"
                        docker port "$CONTAINER_NAME" 5000/tcp
                    '''
                }
            }
        }
    }
}
