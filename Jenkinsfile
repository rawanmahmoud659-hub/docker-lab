
pipeline {
    agent any

    environment {
        APP_NAME = 'app-hello-nti'
        REPO_URL = 'https://github.com/rawanmahmoud659-hub/docker-lab.git'
    }

    stages {
        stage('Getting Repo files') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def buildTag = currentBuild.number.toString()

                    echo "Jenkins build number: ${buildTag}"

                    sh """
                        docker build -t ${APP_NAME}:${buildTag} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def buildTag = currentBuild.number.toString()

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

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    def buildTag = currentBuild.number.toString()
                    def containerName = "${APP_NAME}-main-${buildTag}"

                    sh """
                        if docker container inspect ${containerName} >/dev/null 2>&1; then
                            docker rm -f ${containerName}
                        fi

                        docker run -d \
                            --name ${containerName} \
                            -p 5000:5000 \
                            ${APP_NAME}:${buildTag}

                        docker ps
                    """
                }
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
