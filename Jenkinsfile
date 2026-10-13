pipeline {
    agent any

    environment {
        APP_BASE_NAME = 'app-hello-nti'
        DOCKER_REGISTRY = 'docker.io'
    }

    stages {

        stage('Configure Branch') {
            steps {
                script {
                    // Jenkins Multibranch Pipeline provides BRANCH_NAME
                    env.GIT_BRANCH_NAME = env.BRANCH_NAME

                    env.BRANCH_SLUG = env.GIT_BRANCH_NAME
                        .toLowerCase()
                        .replaceAll('[^a-z0-9_.-]', '-')

                    // Different application name for each branch
                    if (env.GIT_BRANCH_NAME == 'main') {
                        env.APP_NAME = env.APP_BASE_NAME
                    } else if (env.GIT_BRANCH_NAME == 'dev') {
                        env.APP_NAME = "${env.APP_BASE_NAME}-dev"
                    } else if (env.GIT_BRANCH_NAME == 'stg') {
                        env.APP_NAME = "${env.APP_BASE_NAME}-stg"
                    } else {
                        env.APP_NAME =
                            "${env.APP_BASE_NAME}-${env.BRANCH_SLUG}"
                    }

                    env.IMAGE_NAME = "${env.APP_NAME}:${env.BUILD_NUMBER}"
                    env.CONTAINER_NAME =
                        "${env.APP_NAME}-${env.BRANCH_SLUG}-${env.BUILD_NUMBER}"

                    echo "Branch: ${env.GIT_BRANCH_NAME}"
                    echo "App: ${env.APP_NAME}"
                    echo "Image: ${env.IMAGE_NAME}"
                    echo "Container: ${env.CONTAINER_NAME}"
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

        stage('Run Container on Available Port') {
            steps {
                script {
                    // Remove a previous container with the same name, if any
                    sh '''
                        docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
                    '''

                    // Publish container port 5000 on an automatically
                    // allocated available host port
                    sh '''
                        docker run -d \
                            -p 5000 \
                            --name "$CONTAINER_NAME" \
                            "$IMAGE_NAME"
                    '''

                    // Discover the port assigned by Docker
                    env.HOST_PORT = sh(
                        script: """
                            docker port '${env.CONTAINER_NAME}' 5000/tcp |
                            head -n 1 |
                            awk -F: '{print \\$NF}'
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Container started successfully"
                    echo "Branch: ${env.GIT_BRANCH_NAME}"
                    echo "Container: ${env.CONTAINER_NAME}"
                    echo "Host port: ${env.HOST_PORT}"
                    echo "App URL: http://localhost:${env.HOST_PORT}"
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    docker ps --filter "name=$CONTAINER_NAME"
                    docker port "$CONTAINER_NAME"
                    docker logs --tail 20 "$CONTAINER_NAME"
                '''
            }
        }
    }
}
