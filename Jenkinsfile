pipeline {
    agent any

    environment {
        DOCKER_ID = "dimitri15"
        CAST_IMAGE = "cast-service"
        MOVIE_IMAGE = "movie-service"
        IMAGE_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_ID/$CAST_IMAGE:$IMAGE_TAG ./cast-service
                    docker build -t $DOCKER_ID/$MOVIE_IMAGE:$IMAGE_TAG ./movie-service
                    """
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$CAST_IMAGE:$IMAGE_TAG
                    docker push $DOCKER_ID/$MOVIE_IMAGE:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployWithHelm("dev")
                }
            }
        }

        stage('Deploy to qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployWithHelm("qa")
                }
            }
        }

        stage('Deploy to staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployWithHelm("staging")
                }
            }
        }

        stage('Manual Approval for prod') {
            when {
                branch 'master'
            }
            steps {
                script {
                    echo "[INFO] Sur branche master, attente de validation manuelle..."

                    timeout(time: 15, unit: 'MINUTES') {
                        input message: '✅ Voulez-vous déployer en production ?', ok: 'Déployer en PROD'
                    }
                }
            }
        }

        stage('Deploy to prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployWithHelm("prod")
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès pour l’image ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline échoué. Consultez les logs."
        }
    }
}

def deployWithHelm(env) {
    sh """
    echo "[INFO] Déploiement dans l'environnement: ${env}"
    mkdir -p ~/.kube
    cat \$KUBECONFIG > ~/.kube/config

    echo "[INFO] Mise à jour des fichiers Helm..."
    sed -i "s+tag:.*+tag: \\\"${IMAGE_TAG}\\\"+g" charts/cast-service/values-${env}.yaml
    sed -i "s+tag:.*+tag: \\\"${IMAGE_TAG}\\\"+g" charts/movie-service/values-${env}.yaml

    echo "[INFO] Lancement du déploiement Helm"
    helm upgrade --install cast-service charts/cast-service \
        --namespace ${env} \
        -f charts/cast-service/values-${env}.yaml

    helm upgrade --install movie-service charts/movie-service \
        --namespace ${env} \
        -f charts/movie-service/values-${env}.yaml
    """
}

