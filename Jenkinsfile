pipeline {
  agent any

  environment {
    DOCKER_ID = "dorvs"
    CAST_IMAGE = "cast-service"
    MOVIE_IMAGE = "movie-service"
    TAG = "${BRANCH_NAME}-${BUILD_NUMBER}"
    DOCKER_CREDS = credentials("dockerhub-credentials")
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
                    sh '''
                    # Vérifier la connexion au cluster
                    kubectl get ns

                    # Vérifier les droits du ServiceAccount
                    kubectl auth can-i create deployment -n dev

                    # Déployer avec Helm
                    helm upgrade --install app charts \
                        --namespace dev \
                        --create-namespace \
                        --set cast.image.tag=null-25 \
                        --set movie.image.tag=null-25
                    '''
                }
            }
        }

        stage('Deploy Production') {
            when {
                expression { return false } // ou logique pour production
            }
            steps {
                echo "Skipping production due to dev failure"
      }
    }

  } // <-- fin stages
} // <-- fin pipeline
