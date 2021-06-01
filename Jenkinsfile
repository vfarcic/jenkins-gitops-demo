pipeline {
  agent {
    kubernetes {
      yamlFile "jenkins-pod.yaml"
    }
  }
  environment {
    PROJECT = "jenkins-gitops-demo"
    REGISTRY_USER = "vfarcic"
    GITOPS_REPO = "vfarcic/jenkins-gitops-demo"
  }
  stages {
    // stage("Build") {
    //   steps {
    //     container("kaniko") {
    //       sh "/kaniko/executor --context `pwd` --destination vfarcic/jenkins-gitops-demo:latest --destination ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
    //     }
    //   }
    // }
    // stage("Test") {
    //   when { changeRequest target: "master" }
    //   steps {
    //     container("kustomize") {
    //       sh """
    //         set +e
    //         kubectl create namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
    //         set -e
    //         cd kustomize/overlays/preview
    //         kustomize edit set namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
    //         kustomize edit set image $REGISTRY_USER/$PROJECT=$REGISTRY_USER/$PROJECT:${env.BRANCH_NAME.toLowerCase()}-$BUILD_NUMBER
    //         cat ingress.yaml | sed -e "s@host: @host: ${env.BRANCH_NAME.toLowerCase()}@g" | tee ingress.yaml
    //         kustomize build . | kubectl apply --filename -
    //         kubectl --namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()} rollout status deployment jenkins-gitops-demo
    //       """
    //       sh "curl http://${env.BRANCH_NAME.toLowerCase()}$PROJECT.18.198.86.174.nip.io"
    //       sh "kubectl delete namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}"
    //     }
    //   }
    // }
    stage("Deploy") {
      when { branch "master" }
      steps {
        // TODO: Switch to Git image
        container("kustomize") {
          sh """
            mkdir gitops
            git clone https://github.com/$GITOPS_REPO gitops
            cd gitops
            cat production/${PROJECT}.yaml | sed -e "s@${PROJECT}.*@${PROJECT}:${BRANCH_NAME}-${BUILD_NUMBER}@g" | tee production/${PROJECT}.yaml
            git add .
            git commit -m "Updating ${PROJECT} to tag ${BRANCH_NAME}-${BUILD_NUMBER}"
            git push
          """
        }
      }
    }
  }
}
