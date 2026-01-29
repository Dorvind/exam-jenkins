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
    environment {
        DOCKER_TAG = "v.${BUILD_ID}.0"  // <- définit la variable correctement
    }
    when {
        not { branch 'master' }
    }
    steps {
        sh """
        helm upgrade --install app charts \
          --namespace dev \
          --create-namespace \
          --set cast.image.tag=$DOCKER_TAG \
          --set movie.image.tag=$DOCKER_TAG
        """
    }
}

  stage('Deploy Production') {
    when {
        branch 'master'
    }
    input {
        message "Déployer en production ?"
        ok "Déployer"
    }
    steps {
        sh """
        helm upgrade --install app charts \
          --namespace prod \
          --create-namespace \
          --set cast.image.tag=$DOCKER_TAG \
          --set movie.image.tag=$DOCKER_TAG
        """
  }
 }
}
