pipeline {
  agent any

  environment {
    REGISTRY_URL = '352708296901.dkr.ecr.us-east-1.amazonaws.com/eliasrepo'
    ECR_REGION = 'us-east-1'
    K8S_NAMESPACE = 'devops-alfnar-k8s-eliasNM'
    K8S_CLUSTER_NAME = 'devops-alfnar-k8s'
    K8S_CLUSTER_REGION = 'eu-north-1'
  }

  stages {
    stage('MNIST Predictor - building....'){
        when { branch "master" }
        steps {
            sh '''
            IMAGE="mnist-predictor:0.0.${BUILD_NUMBER}"
            cd ml_model
            aws ecr get-login-password --region $ECR_REGION | docker login --username AWS --password-stdin ${REGISTRY_URL}
            docker build -t ${IMAGE} .
            docker tag ${IMAGE} ${REGISTRY_URL}/${IMAGE}
            docker push ${REGISTRY_URL}/${IMAGE}
            '''
        }
    }

    stage('MNIST Predictor - deplooy'){
        when { branch "master" }
        steps {
            sh '''
            cd infra/k8s
            IMG_NAME=mnist-predictor:0.0.${BUILD_NUMBER}

            # replace registry url and image name placeholders in yaml
            sed -i "s/{{REGISTRY_URL}}/$REGISTRY_URL/g" mnist-predictor.yaml
            sed -i "s/{{K8S_NAMESPACE}}/$K8S_NAMESPACE/g" mnist-predictor.yaml
            sed -i "s/{{IMG_NAME}}/$IMG_NAME/g" mnist-predictor.yaml

            # get kubeconfig creds
            aws eks --region $K8S_CLUSTER_REGION update-kubeconfig --name $K8S_CLUSTER_NAME

            # apply to your namespace
            kubectl apply -f mnist-predictor.yaml -n $K8S_NAMESPACE
            '''
        }
    }
  }
}


