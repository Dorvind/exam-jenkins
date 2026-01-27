pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "dockerhub_user"
        IMAGE_NAME = "app_name"
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/Dorvind/exam-jenkins.git'
            }
        }

        stage('Tests') {
            steps {
                sh 'docker-compose up -d'
                sh 'docker-compose exec app pytest'
                sh 'docker-compose down'
            }
        }

        stage('Build Image') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:${BRANCH_NAME}-${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Image') {
            steps {
                sh """
                echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                docker push $DOCKERHUB_USER/$IMAGE_NAME:${BRANCH_NAME}-${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy Dev / QA / Staging') {
            when {
                not { branch 'master' }
            }
            steps {
                sh """
                helm upgrade --install app helm-chart \
                --namespace ${BRANCH_NAME} \
                --set image.tag=${BRANCH_NAME}-${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy Production') {
            when {
                branch 'master'
            }
            input {
                message "Déployer en PRODUCTION ?"
                ok "Déployer"
            }
            steps {
                sh """
                helm upgrade --install app helm-chart \
                --namespace prod \
                --set image.tag=master-${BUILD_NUMBER}
                """
            }
        }
    }
}
