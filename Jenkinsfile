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
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                sh '''
                    docker build -t docker.io/'"$DOCKER_USER"'/demo-app:$BRANCH_NAME .
                    docker push docker.io/'"$DOCKER_USER"'/demo-app:$BRANCH_NAME
                '''
            }
        }

        stage("Deploy To Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KCFG')]) {
                    sh '''
                        mkdir -p ./kube
                        cp $KCFG ./kube/config
                        chmod 600 ./kube/config

                        sed -i "s|IMAGE|docker.io/'"$DOCKER_USER"'/demo-app:$BRANCH_NAME|g" k8s/deployment.yaml

                        kubectl --kubeconfig=./kube/config apply -f k8s/deployment.yaml -n $BRANCH_NAME
                        kubectl --kubeconfig=./kube/config get pods -n $BRANCH_NAME
                    '''
                }
            }
        }
    }
}
