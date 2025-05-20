pipeline {
    agent any

    environment {
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_USER = "dimitri15"
    }

    stages {
        stage('Build & Push Images') {
            steps {
                script {
                    docker.build("${DOCKERHUB_USER}/cast-service:${IMAGE_TAG}", './cast-service')
                    docker.build("${DOCKERHUB_USER}/movie-service:${IMAGE_TAG}", './movie-service')

                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        docker.image("${DOCKERHUB_USER}/cast-service:${IMAGE_TAG}").push()
                        docker.image("${DOCKERHUB_USER}/movie-service:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                script {
                    deployToEnv('dev')
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    deployToEnv('qa')
                }
            }
        }

        stage('Deploy to STAGING') {
            steps {
                script {
                    deployToEnv('staging')
                }
            }
        }

        stage('Manual Approval for PROD') {
            when {
                allOf {
                    branch 'master'
                }
            }
            steps {
                input message: "✅ Déployer manuellement en PROD ?"
            }
        }

        stage('Deploy to PROD') {
            when {
                allOf {
                    branch 'master'
                }
            }
            steps {
                script {
                    deployToEnv('prod')
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès."
        }
        failure {
            echo "❌ Pipeline échoué."
        }
    }
}

def deployToEnv(envName) {
    sh """
    echo "🔧 Mise à jour des fichiers Helm pour ${envName}"
    sed -i 's|tag:.*|tag: "${IMAGE_TAG}"|' charts/cast-service/values-${envName}.yaml
    sed -i 's|tag:.*|tag: "${IMAGE_TAG}"|' charts/movie-service/values-${envName}.yaml

    echo "🚀 Déploiement dans le namespace ${envName}"
    helm upgrade --install cast-service charts/cast-service \
        --namespace ${envName} \
        -f charts/cast-service/values-${envName}.yaml

    helm upgrade --install movie-service charts/movie-service \
        --namespace ${envName} \
        -f charts/movie-service/values-${envName}.yaml
    """
}

