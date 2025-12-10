@Library('my-shared-lib') _

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"

        // Deployment name based on branch
        DEPLOY_NAME = (env.BRANCH_NAME == "main") ? "k8s-app" : "demo-deploy"
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

        stage("Verify Rollout Status") {
            steps {
                script {
                    try {
                        echo "⏳ Checking rollout for $DEPLOY_NAME in namespace ${env.BRANCH_NAME}"
                        sh """
                        kubectl rollout status deployment/${DEPLOY_NAME} \
                        -n ${env.BRANCH_NAME} --timeout=60s
                        """
                        echo "🎉 Deployment Successful!"
                    } catch (err) {
                        echo "❌ Deployment Failed — Rolling Back!"
                        sh "kubectl rollout undo deployment/${DEPLOY_NAME} -n ${env.BRANCH_NAME}"
                        error("Rollback Triggered — Deployment Failed")
                    }
                }
            }
        }
    }
}
