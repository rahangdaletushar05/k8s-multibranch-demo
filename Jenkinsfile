pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
    }

    stages {

        stage("Checkout") {
            steps {
                echo "Branch Running → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage("Docker Login") {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u ${DOCKER_USER} --password-stdin
                    """
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                sh """
                    docker build -t docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME} .
                    docker push docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}
                """
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBE_CONFIG')]) {
                    sh """
                        mkdir -p /tmp/kube
                        cp \$KUBE_CONFIG /tmp/kube/config

                        # Replace IMAGE tag dynamically per branch
                        sed -i "s|IMAGE|docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}|g" k8s/deployment.yaml

                        # Deploy per branch namespace
                        kubectl --kubeconfig=/tmp/kube/config apply -f k8s/deployment.yaml -n ${env.BRANCH_NAME}
                        kubectl --kubeconfig=/tmp/kube/config get pods -n ${env.BRANCH_NAME}
                    """
                }
            }
        }
    }
}
