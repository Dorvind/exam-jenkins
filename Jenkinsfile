pipeline {
  agent any

  environment {
    DOCKER_ID = "dorvs"
    CAST_IMAGE = "cast-service"
    MOVIE_IMAGE = "movie-service"
    TAG = "${BRANCH_NAME}-${BUILD_NUMBER}"
    DOCKER_CREDS = credentials("dockerhub-credentials")
    KUBECONFIG = credentials('kubeconfig-dev') // fichier kubeconfig avec token ServiceAccount
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

          # Vérifier que le ServiceAccount a les droits
          kubectl auth can-i create deployment -n dev

          # Déployer avec Helm en utilisant les tags construits dynamiquement
          helm upgrade --install app charts \
              --namespace dev \
              --create-namespace \
              --set cast.image.tag=$TAG \
              --set movie.image.tag=$TAG
          '''
        }
      }
    }

    stage('Deploy Production') {
      when {
        expression { return false } // Logique pour production
      }
      steps {
        echo "Skipping production due to dev failure"
      }
    }

  } // fin stages
} // fin pipeline
