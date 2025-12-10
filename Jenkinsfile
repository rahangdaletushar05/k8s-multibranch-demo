@Library('my-shared-lib') _   // MUST match Jenkins → Global Library name

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
    }

    stages {

        stage("Checkout") {
            steps {
                echo "🔹 Running Branch → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage("Docker Build & Push") {
            steps {
                sh """
                docker build -t docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME} .
                docker push docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}
                """
            }
        }

        stage("Deploy To Kubernetes") {
            steps {
                k8sDeploy(
                    image: "docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}",
                    namespace: env.BRANCH_NAME,
                    deployFile: "k8s/deployment.yaml",
                    credential: "k8s-config"
                )
            }
        }

        /* 🔥 NEW STAGE → Wait for Deployment to Get Ready  */
        stage("Verify Rollout Status") {
            steps {
                sh """
                kubectl rollout status deployment/demo-app \
                -n ${env.BRANCH_NAME} --timeout=60s
                """
            }
        }
    }
}
