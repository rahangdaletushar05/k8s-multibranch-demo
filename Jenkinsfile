pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
        KUBE_NAMESPACE = "${env.BRANCH_NAME}"   // Namespace = branch name
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

        stage("Deploy To Kubernetes") {
            when {
                expression { return env.BRANCH_NAME in ["dev","main","stage","prod"] }
            }
            steps {
                withCredentials([file(credentialsId: 'kube-config', variable: 'KCFG')]) {
                    sh """
                        mkdir -p /tmp/kube
                        cp \$KCFG /tmp/kube/config

                        sed -i "s|IMAGE|docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}|g" k8s/deployment.yaml

                        kubectl --kubeconfig=/tmp/kube/config apply -f k8s/deployment.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=/tmp/kube/config get pods -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }
    }
}
