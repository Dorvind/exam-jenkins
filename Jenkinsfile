pipeline {
    agent any

    environment {
        DOCKER_ID = "dorvs"
        CAST_IMAGE = "cast-service"
        MOVIE_IMAGE = "movie-service"
        TAG = "${BRANCH_NAME}-${BUILD_NUMBER}"
        DOCKER_CREDS = credentials("dockerhub-credentials")
        // KUBECONFIG utilisé si Jenkins est hors cluster (Secret File dans Jenkins)
        KUBECONFIG = credentials('kubeconfig-dev')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t $DOCKER_ID/$CAST_IMAGE:$TAG cast-service
                docker build -t $DOCKER_ID/$MOVIE_IMAGE:$TAG movie-service
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                sh '''
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                docker push $DOCKER_ID/$CAST_IMAGE:$TAG
                docker push $DOCKER_ID/$MOVIE_IMAGE:$TAG
                '''
            }
        }

        stage('Deploy Dev / QA / Staging') {
    steps {
        script {
            echo "Deploying to dev namespace..."

            withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBECONFIG')]) {
                sh '''
                echo "Using kubeconfig from Jenkins credentials"
                export KUBECONFIG=$KUBECONFIG

                # Vérification du kubeconfig
                head -n 5 $KUBECONFIG

                # Vérifier l’accès Kubernetes
                kubectl get ns
                kubectl auth can-i create deployment -n dev

                # Déploiement Helm
                helm upgrade --install app charts \
                    --namespace dev \
                    --create-namespace \
                    --set cast.image.tag=$TAG \
                    --set movie.image.tag=$TAG
                '''
            }
        }
    }
}


        stage('Deploy Production') {
            when {
                expression { return false } // modifier selon logique Prod
            }
            steps {
                echo "Skipping production deployment"
            }
        }

    } // fin stages
}
