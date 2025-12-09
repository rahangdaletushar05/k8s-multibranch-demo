pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/YOUR_DOCKERHUB_USER"    // change this
        IMAGE_NAME = "demo-app"
        KUBE_CONFIG = credentials('kubeconfig-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Build & Push") {
            steps {
                sh """
                docker build -t $REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME} .
                docker push $REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    if (env.BRANCH_NAME == "prod") {
                        input message: "Approve Production Deployment?"
                    }
                    sh """
                    export KUBECONFIG=${KUBE_CONFIG}
                    kubectl apply -f k8s/deployment.yaml
                    """
                }
            }
        }
    }
}
