pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/YOUR_DOCKERHUB_USERNAME"
        IMAGE_NAME = "k8s-demo-app"
        KUBE_CONFIG = credentials('kubeconfig-creds')   // upload kubeconfig in Jenkins
        DOCKER_CREDS = credentials('dockerhub-creds')   // add docker creds in Jenkins
    }

    stages {

        stage("Git Clone") {
            steps {
                checkout scm
                echo "Branch -> ${env.BRANCH_NAME}"
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

        stage("Apply K8s Manifests") {
            steps {
                script {
                    // store kubeconfig to Jenkins agent
                    sh """
                    mkdir -p ~/.kube
                    echo '${KUBE_CONFIG}' > ~/.kube/config
                    """

                    sh """
                    kubectl set image deployment/k8s-app k8s-container=$REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME} -n default || \
                    kubectl apply -f k8s/
                    """
                }
            }
        }

        stage("Rollout Status Check") {
            steps {
                sh """
                kubectl rollout status deployment/k8s-app -n default --timeout=60s
                """
            }
        }
    }
}

