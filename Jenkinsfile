pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
        IMAGE_NAME = "demo-app"
    }

    stages {

        stage("Checkout") {
            steps {
                echo "Branch Running → ${env.GIT_BRANCH}"
                checkout scm
            }
        }

        stage("Docker Login") {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u ${DOCKER_USER} --password-stdin
                    """
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                sh """
                    docker build -t docker.io/${DOCKER_USER}/${IMAGE_NAME}:dev .
                    docker push docker.io/${DOCKER_USER}/${IMAGE_NAME}:dev
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBE_CONFIG')]) {
                    sh """
                        mkdir -p /tmp/.kube
                        cp \$KUBE_CONFIG /tmp/.kube/config

                        sed -i 's|IMAGE|docker.io/${DOCKER_USER}/${IMAGE_NAME}:dev|g' k8s/deployment.yaml

                        kubectl --kubeconfig=/tmp/.kube/config apply -f k8s/deployment.yaml -n dev
                    """
                }
            }
        }
    }
}
