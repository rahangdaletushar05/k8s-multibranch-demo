@Library('my-shared-lib') _   // Shared Library

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
        DEPLOY_NAME = "demo-deploy"   // your deployment name in cluster
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

        /* 🔥 Final Stage — Rollout Verification + Auto Rollback */
        stage("Verify Rollout Status") {
            steps {
                script {
                    try {
                        echo "⏳ Checking deployment rollout..."
                        sh """
                        kubectl rollout status deployment/${DEPLOY_NAME} \
                        -n ${env.BRANCH_NAME} --timeout=60s
                        """
                        echo "🎉 Deployment Successful!"
                    } catch (err) {
                        echo "❌ Deployment Failed — Rolling Back!"
                        sh """
                        kubectl rollout undo deployment/${DEPLOY_NAME} \
                        -n ${env.BRANCH_NAME}
                        """
                        error("Pipeline Failed — Rollback Triggered")
                    }
                }
            }
        }
    }
}
