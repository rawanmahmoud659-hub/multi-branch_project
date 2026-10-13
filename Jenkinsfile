stage('Run Docker Container') {
    steps {
        script {
            // Use the branch detected by Jenkins
            env.GIT_BRANCH_NAME = env.BRANCH_NAME

            if (env.GIT_BRANCH_NAME == 'main') {
                env.APP_NAME = 'app-hello-nti'
            } else if (env.GIT_BRANCH_NAME == 'dev') {
                env.APP_NAME = 'app-hello-nti-dev'
            } else if (env.GIT_BRANCH_NAME == 'stg') {
                env.APP_NAME = 'app-hello-nti-stg'
            } else {
                env.APP_NAME = "app-hello-nti-${env.GIT_BRANCH_NAME}"
            }

            env.CONTAINER_NAME =
                "${env.APP_NAME}-${env.BUILD_NUMBER}"

            sh '''
                docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                docker run -d \
                    -p 5000 \
                    --name "$CONTAINER_NAME" \
                    "$APP_NAME:$BUILD_NUMBER"

                docker ps --filter "name=$CONTAINER_NAME"
                docker port "$CONTAINER_NAME" 5000/tcp
            '''
        }
    }
}
