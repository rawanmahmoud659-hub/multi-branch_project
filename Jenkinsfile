pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/MohamedMagdy840/jenkins-repo.git'
        REGISTRY = 'docker.io'
    }

    stages {
        stage('Getting Repo Files') {
            steps {
                git branch: "${env.BRANCH_NAME ?: 'main'}",
                    credentialsId: 'github',
                    url: "${REPO_URL}"
            }
        }

        stage('Detect Branch and Configure App') {
            steps {
                script {
                    // Detect the current Git branch
                    env.GIT_BRANCH_NAME = sh(
                        script: 'git rev-parse --abbrev-ref HEAD',
                        returnStdout: true
                    ).trim()

                    // Make the branch name safe for Docker names and tags
                    env.BRANCH_SLUG = env.GIT_BRANCH_NAME
                        .toLowerCase()
                        .replaceAll('[^a-z0-9_.-]', '-')

                    // Change the app name based on the branch
                    if (env.GIT_BRANCH_NAME == 'main') {
                        env.APP_NAME = 'new-app'
                    } else if (env.GIT_BRANCH_NAME == 'dev') {
                        env.APP_NAME = 'new-app-dev'
                    } else {
                        env.APP_NAME = "new-app-${env.BRANCH_SLUG}"
                    }

                    env.IMAGE_NAME = "${env.APP_NAME}:${env.BUILD_NUMBER}"
                    env.CONTAINER_NAME =
                        "${env.APP_NAME}-${env.BRANCH_SLUG}-${env.BUILD_NUMBER}"

                    echo "Git branch: ${env.GIT_BRANCH_NAME}"
                    echo "Application: ${env.APP_NAME}"
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

        stage('Find Available Port and Run Container') {
            steps {
                script {
                    // Remove this exact container if it already exists
                    sh '''
                        docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
                    '''

                    // Docker automatically allocates a host port
                    sh '''
                        docker run -d \
                            -p 5000 \
                            --name "$CONTAINER_NAME" \
                            "$IMAGE_NAME"
                    '''

                    // Display the assigned host port
                    env.HOST_PORT = sh(
                        script: """
                            docker port '${env.CONTAINER_NAME}' 5000/tcp |
                            head -n 1 |
                            awk -F: '{print \\$NF}'
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Container: ${env.CONTAINER_NAME}"
                    echo "Application URL: http://localhost:${env.HOST_PORT}"
                }
            }
        }

        stage('Verify Container') {
            steps {
                sh '''
                    docker ps --filter "name=$CONTAINER_NAME"
                    docker port "$CONTAINER_NAME"
                '''
            }
        }
    }
}
