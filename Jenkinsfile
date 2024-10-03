pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git changelog: false, credentialsId: 'git-cred', poll: false, url: 'https://github.com/skommana04/AuthMicroService.git'
            }
        }

        stage('Docker Build & Test') {
            steps {
                script {
                    def imageVersion = "v${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker build -t medhakommana/auth931:${imageVersion} -f Dockerfile ."
                        sh "docker push medhakommana/auth931:${imageVersion}"
                    }
                }
            }
        }

        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "authdeploy"
                GIT_USER_NAME = "skommana04"
            }
            steps {
                git changelog: false, credentialsId: 'git-cred', poll: false, url: 'https://github.com/skommana04/authdeploy.git'
                withCredentials([string(credentialsId: 'git-cred', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def imageVersion = "v${env.BUILD_NUMBER}"
                        sh '''
                            imageTag=$(grep "image:" authms-deployment.yml | awk -F ":" '{print $3}')
                            echo "Old Image Tag: $imageTag"
                            sed -i "s/auth931:${imageTag}/auth931:${imageVersion}/g" authms-deployment.yml
                            git add authms-deployment.yml
                            git commit -am "Updated deployment Image to version ${imageVersion}"
                            git push https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
