pipeline {
    agent any

    environment {
        DOCKER_IMAGE  = "zree7/fastapi-app"
        GIT_REPO_NAME = "fastapi-app"
        GIT_USER_NAME = "Zree77"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Zree77/fastapi-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarscanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=fastapi-gitops-app \
                              -Dsonar.projectName='FastAPI GitOps App' \
                              -Dsonar.sources=. \
                              -Dsonar.python.version=3.11 \
                              -Dsonar.sourceEncoding=UTF-8
                        """
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'

                    def dockerImage = docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}")

                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        'docker-cred'
                    ) {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Update Deployment File') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'github',
                usernameVariable: 'GITHUB_USERNAME',
                passwordVariable: 'GITHUB_TOKEN'
            )
        ]) {
            sh '''
                git config user.email "zreeprojects@gmail.com"
                git config user.name "${GIT_USER_NAME}"

                sed -i "s|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g" k8s/deployment.yaml

                git add k8s/deployment.yaml

                git commit \
                  -m "Update FastAPI app image tag to ${BUILD_NUMBER} [skip ci]" \
                  || echo "No changes to commit"

                git push \
                  https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git \
                  HEAD:main
            '''
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
