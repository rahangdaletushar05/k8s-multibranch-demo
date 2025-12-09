pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/tusharrahangdale"           // your DockerHub user
        IMAGE_NAME = "demo-app"                           // keep lowercase
        DOCKER_CREDS = credentials('dockerhub-creds')
        KUBE_CONFIG  = credentials('kubeconfig-creds')
    }

    stages {

        stage("Checkout") {
            steps {
                checkout scm
                echo "Branch Running → ${env.BRANCH_NAME}"
            }
        }

        stage("Docker Login") {
            steps {
                sh """
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                """
            }
        }

        stage("Build & Push Docker Image") {
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
                    sh "mkdir -p ~/.kube"
                    writeFile file: "~/.kube/config", text: KUBE_CONFIG

                    sh """
                    sed -i 's|IMAGE|$REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}|g' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml -n dev
                    kubectl apply -f k8s/service.yaml -n dev
                    kubectl rollout status deployment/k8s-app -n dev
                    """
                }
            }
        }
    }
}
