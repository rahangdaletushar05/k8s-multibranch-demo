pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
    }

    stages {

        stage("Checkout") {
            steps {
                echo "🔹 Branch Running → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage("Docker Login") {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u $DOCKER_USER --password-stdin'
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

        stage("Deploy To Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KCFG')]) {
                    sh """
                        mkdir -p /tmp/kube
                        cp \$KCFG /tmp/kube/config

                        kubectl --kubeconfig=/tmp/kube/config set image deployment/k8s-app \
                        k8s-container=docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME} -n ${env.BRANCH_NAME}

                        kubectl --kubeconfig=/tmp/kube/config get pods -n ${env.BRANCH_NAME}
                    """
                }
            }
        }
    }
}
