pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/tusharrahangdale"          // Your DockerHub Username
        IMAGE_NAME = "k8s-demo-app"
        DOCKER_CREDS = credentials('dockerhub-creds')    // Add in Jenkins Credentials
        KUBE_CONFIG = credentials('kubeconfig-creds')    // Upload kubeconfig in Jenkins
    }

    stages {

        stage("Git Clone") {
            steps {
                checkout scm
                echo "Building from branch → ${env.BRANCH_NAME}"
            }
        }

        stage("Docker Login") {
            steps {
                sh """
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                """
            }
        }

        stage("Docker Build + Push") {
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
                    // Store kubeconfig for kubectl access
                    sh """
                    mkdir -p ~/.kube
                    echo '${KUBE_CONFIG}' > ~/.kube/config
                    """

                    // Deploy with new image per branch
                    sh """
                    sed -i 's|IMAGE|$REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}|g' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }

        stage("Rollout Status Check") {
            steps {
                sh """
                kubectl rollout status deployment/k8s-app --timeout=60s
                """
            }
        }
    }
}
