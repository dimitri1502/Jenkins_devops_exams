pipeline {
  environment {
    DOCKER_ID = "dimitri15"
    MOVIE_IMAGE = "movie_service"
    CAST_IMAGE = "cast_service"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }

  agent any

  stages {
    stage('Docker Build') {
      steps {
        script {
          sh '''
          docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
          docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
          '''
        }
      }
    }

    stage('Test Acceptance') {
      steps {
        script {
          sh '''
          echo "Tests d'acceptance à ajouter ici (curl, pytest, etc.)"
          '''
        }
      }
    }

    stage('Docker Push') {
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
          docker login -u $DOCKER_ID -p $DOCKER_PASS
          docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
          docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
          '''
        }
      }
    }

    stage('Helm Deploy - Dev') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          mkdir -p .kube
          cat $KUBECONFIG > .kube/config
          cp charts/values.yaml values-dev.yaml
          sed -i "s/tag:.*/tag: ${DOCKER_TAG}/g" values-dev.yaml
          helm upgrade --install movie-service charts --values values-dev.yaml --namespace dev --create-namespace
          '''
        }
      }
    }

    stage('Helm Deploy - QA') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          mkdir -p .kube
          cat $KUBECONFIG > .kube/config
          cp charts/values.yaml values-qa.yaml
          sed -i "s/tag:.*/tag: ${DOCKER_TAG}/g" values-qa.yaml
          helm upgrade --install movie-service charts --values values-qa.yaml --namespace qa --create-namespace
          '''
        }
      }
    }

    stage('Helm Deploy - Staging') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          mkdir -p .kube
          cat $KUBECONFIG > .kube/config
          cp charts/values.yaml values-staging.yaml
          sed -i "s/tag:.*/tag: ${DOCKER_TAG}/g" values-staging.yaml
          helm upgrade --install movie-service charts --values values-staging.yaml --namespace staging --create-namespace
          '''
        }
      }
    }

    stage('Helm Deploy - Prod') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Deploy to production?', ok: 'Yes'
        }

        script {
          sh '''
          mkdir -p .kube
          cat $KUBECONFIG > .kube/config
          cp charts/values.yaml values-prod.yaml
          sed -i "s/tag:.*/tag: ${DOCKER_TAG}/g" values-prod.yaml
          helm upgrade --install movie-service charts --values values-prod.yaml --namespace prod --create-namespace
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline executed successfully. Image tag: ${DOCKER_TAG}"
    }
    failure {
      echo "Pipeline failed. Check logs."
    }
  }
}
