pipeline {
  agent any
  parameters {
    choice(name: 'NEW_COLOR', choices: ['blue', 'green'], description: 'Color to deploy')
  }
  environment {
    IMAGE        = "adamyasrivastava/mysite"
    DEPLOY_COLOR = "${params.NEW_COLOR ?: 'blue'}"
    TAG          = "${params.NEW_COLOR ?: 'blue'}-${BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Push Image') {
      steps {
        sh 'sed -i "s/COLOR_PLACEHOLDER/$DEPLOY_COLOR/g" index.html'
        sh 'docker build -t $IMAGE:$TAG .'
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
          usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo $P | docker login -u $U --password-stdin'
          sh 'docker push $IMAGE:$TAG'
        }
      }
    }
    stage('Deploy New Color') {
      steps {
        sh 'kubectl get svc bgsite-svc || kubectl apply -f k8s/service.yaml'
        sh 'sed "s/COLOR/$DEPLOY_COLOR/g; s/IMAGE_TAG/$TAG/g" k8s/deploy-template.yaml > k8s/deploy.yaml'
        sh 'kubectl apply -f k8s/deploy.yaml'
        sh 'kubectl rollout status deployment/bgsite-$DEPLOY_COLOR'
      }
    }
    stage('Approval') {
      steps {
        input message: "Switch live traffic to ${env.DEPLOY_COLOR}?", ok: 'Switch'
      }
    }
    stage('Switch Traffic') {
      steps {
        sh 'kubectl set selector svc bgsite-svc app=bgsite,color=$DEPLOY_COLOR'
        sh 'kubectl get svc bgsite-svc -o wide'
      }
    }
  }
}
