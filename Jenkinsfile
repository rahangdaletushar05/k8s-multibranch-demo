pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "tusharrahangdale/demo-app"
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
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u tussrrahangdale --password-stdin
                    """
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                sh """
                docker build -t ${REGISTRY}/${IMAGE_NAME}:${env.BRANCH_NAME} .
                docker push ${REGISTRY}/${IMAGE_NAME}:${env.BRANCH_NAME}
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBE_CONFIG')]) {
                    sh """
                    mkdir -p ~/.kube
                    cp \$KUBE_CONFIG ~/.kube/config

                    sed -i s|IMAGE|${REGISTRY}/${IMAGE_NAME}:${env.BRANCH_NAME}|g k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml -n dev --validate=false
                    kubectl rollout status deployment/demo-app -n dev --timeout=60s
                    """
                }
            }
        }
    }
}
