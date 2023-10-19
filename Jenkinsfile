pipeline {
    agent any

    environment {
        // Dynamically get the branch name that triggered the build
        BRANCH_NAME = "${GIT_BRANCH.split('/').last()}"
        // Define Docker image name based on the branch
        IMAGE_NAME = "9989228601/my-web-app:${BRANCH_NAME}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Check out code from the branch that triggered this build
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${IMAGE_NAME} ."

                    // Log in to DockerHub using Jenkins-stored credentials and push the image
                    withCredentials([usernamePassword(credentialsId: '20d12623-70d2-43b8-ac30-6aef22bfefcb', passwordVariable: 'DOCKER_HUB_PWD', usernameVariable: 'DOCKER_HUB_USER')]) {
                        sh "docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_PWD"
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Deploy to Dev Server') {
            when {
                expression { BRANCH_NAME == 'dev' }
            }
            steps {
                // Use Ansible for deployment to Dev
                sh "ansible-playbook -i ${WORKSPACE}/multi-branch_dev/inventory.ini ${WORKSPACE}/multi-branch_dev/deploy.yml -e branch=${BRANCH_NAME}"
            }
        }

        stage('Deploy to Prod Server') {
            when {
                expression { BRANCH_NAME == 'prod' }
            }
            steps {
                // Use Ansible for deployment to Prod
                sh "ansible-playbook -i ${WORKSPACE}/multi-branch_dev/inventory.ini ${WORKSPACE}/multi-branch_dev/deploy.yml -e branch=${BRANCH_NAME}"
            }
        }
    }
}
