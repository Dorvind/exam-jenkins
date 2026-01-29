pipeline {
    agent any

    environment {
        DOCKER_ID = "dorvs"
        DOCKER_IMAGE = "dora-examen"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Docker Build') {
            steps {
                sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    sleep 6
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                '''
            }
        }

        stage('Test Acceptance') {
            steps {
                sh 'curl localhost'
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deployments') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    def namespaces = ["dev", "staging"]
                    for (ns in namespaces) {
                        sh """
                            rm -Rf .kube
                            mkdir .kube
                            cat $KUBECONFIG > .kube/config
                            cp fastapi/values.yaml values.yml
                            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                            helm upgrade --install app fastapi --values=values.yml --namespace $ns --create-namespace
                        """
                    }
                }
            }
        }

        stage('Deploy Production') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy in production?', ok: 'Yes'
                }
                sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod --create-namespace
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker containers"
            sh 'docker rm -f jenkins || true'
        }
        success {
            echo "Pipeline finished successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
