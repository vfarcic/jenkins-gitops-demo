pipeline {
  agent {
    kubernetes {
      yamlFile "jenkins-pod.yaml"
    }
  }
  environment {
    PROJECT = "jenkins-demo"
    REGISTRY_USER = "vfarcic"
  }
  stages {
    stage("Build") {
      steps {
        container("kaniko") {
          sh "/kaniko/executor --context `pwd` --destination vfarcic/jenkins-demo:latest --destination ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
        }
      }
    }
    stage("Test") {
      when { changeRequest target: "master" }
      steps {
        container("kustomize") {
          sh """
            set +e
            kubectl create namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
            set -e
            cd kustomize/overlays/preview
            kustomize edit set namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
            kustomize edit set image $REGISTRY_USER/$PROJECT=$REGISTRY_USER/$PROJECT:${env.BRANCH_NAME.toLowerCase()}-$BUILD_NUMBER
            cat ingress.yaml | sed -e "s@host: @host: ${env.BRANCH_NAME.toLowerCase()}@g" | tee ingress.yaml
            kustomize build . | kubectl apply --filename -
            kubectl --namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()} rollout status deployment jenkins-demo
          """
          sh "curl http://${env.BRANCH_NAME.toLowerCase()}$PROJECT.acme.com"
          sh "kubectl delete namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}"
        }
      }
    }
    stage("Deploy") {
      when { branch "master" }
      steps {
        container("kustomize") {
          sh """
            cd kustomize/overlays/production
            kustomize edit set image ${REGISTRY_USER}/${PROJECT}=${REGISTRY_USER}/${PROJECT}:$BRANCH_NAME-${BUILD_NUMBER}
            kustomize build . | kubectl apply --filename -
          """
        }
      }
    }
  }
}
