pipeline {
    agent any

    environment {
        DOCKER_USER = 'gouns'
        DOCKER_PASS = credentials('DOCKER_HUB_PASS')
        IMAGE_MOVIE = "${DOCKER_USER}/movie-service"
        IMAGE_CAST  = "${DOCKER_USER}/cast-service"
        IMAGE_TAG   = "v${BUILD_NUMBER}"
        KUBECONFIG  = credentials('config')
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
                    docker build -t ${IMAGE_MOVIE}:${IMAGE_TAG} ./movie-service
                    docker build -t ${IMAGE_CAST}:${IMAGE_TAG}  ./cast-service
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_MOVIE}:${IMAGE_TAG}
                    docker push ${IMAGE_CAST}:${IMAGE_TAG}
                    docker logout
                '''
            }
        }

        stage('Deploy - dev') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install movie-service ./charts \
                        --namespace dev \
                        --set image.repository=${IMAGE_MOVIE} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30007

                    helm upgrade --install cast-service ./charts \
                        --namespace dev \
                        --set image.repository=${IMAGE_CAST} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30008
                '''
            }
        }

        stage('Deploy - qa') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install movie-service ./charts \
                        --namespace qa \
                        --set image.repository=${IMAGE_MOVIE} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30007

                    helm upgrade --install cast-service ./charts \
                        --namespace qa \
                        --set image.repository=${IMAGE_CAST} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30008
                '''
            }
        }

        stage('Deploy - staging') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install movie-service ./charts \
                        --namespace staging \
                        --set image.repository=${IMAGE_MOVIE} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30007

                    helm upgrade --install cast-service ./charts \
                        --namespace staging \
                        --set image.repository=${IMAGE_CAST} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30008
                '''
            }
        }

        stage('Deploy - prod') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Déployer en production ?', ok: 'Confirmer'
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install movie-service ./charts \
                        --namespace prod \
                        --set image.repository=${IMAGE_MOVIE} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30007

                    helm upgrade --install cast-service ./charts \
                        --namespace prod \
                        --set image.repository=${IMAGE_CAST} \
                        --set image.tag=${IMAGE_TAG} \
                        --set service.nodePort=30008
                '''
            }
        }

    }

    post {
        success {
            echo "Pipeline terminé avec succès - Image tag: ${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline en échec - vérifier les logs"
        }
    }
}
