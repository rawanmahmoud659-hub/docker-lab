
pipeline {
    agent any

    environment {
        APP_NAME = 'app-hello-nti'
        REPO_URL = 'https://github.com/rawanmahmoud659-hub/docker-lab.git'
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: 'main',
                    credentialsId: '4607176d-d42f-4dc9-8107-466a24a71408',
                    url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "$APP_NAME:$docker-tken" .'
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
                        set +x
                        echo "$DOCKER_PASSWORD" |
                            docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker tag "$APP_NAME:$BUILD_NUMBER" \
                            "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"

                        docker push "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    CONTAINER_NAME="${APP_NAME}-main-${BUILD_NUMBER}"

                    if docker container inspect "$CONTAINER_NAME" >/dev/null 2>&1; then
                        docker rm -f "$CONTAINER_NAME"
                    fi

                    docker run -d \
                        --name "$CONTAINER_NAME" \
                        -p 5000:5000 \
                        "$APP_NAME:$BUILD_NUMBER"

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
