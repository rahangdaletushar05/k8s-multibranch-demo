pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/tusharrahangdale"
        IMAGE_NAME = "demo-app"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Branch Running → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    """
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                sh """
                docker build -t $REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME} .
                docker push $REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBE_CONFIG')]) {
                    sh """
                    mkdir -p ~/.kube
                    cp $KUBE_CONFIG ~/.kube/config

                    # Replace image dynamically
                    sed -i s|IMAGE|$REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}|g k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml -n dev --validate=false
                    kubectl rollout status deployment/demo-app -n dev
                    """
                }
            }
        }
    }
}
