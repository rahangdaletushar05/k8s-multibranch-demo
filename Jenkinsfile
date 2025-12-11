@Library('my-shared-lib') _

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

        /* 🔐 Docker Login (Updated with dockerhub-creds-2) */
        stage("Docker Login") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds-2',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    """
                }
            }
        }

        /* 🛠 Build + Push */
        stage("Docker Build & Push") {
            steps {
                sh """
                docker build -t docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME} .
                docker push docker.io/${DOCKER_USER}/demo-app:${env.BRANCH_NAME}
                """
            }
        }

        /* 🚀 Deploy To Kubernetes */
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

        /* 🔎 Verify Rollout */
        stage("Verify Rollout Status") {
            steps {
                script {

                    def DEPLOY_NAME = (env.BRANCH_NAME == "main") ? "k8s-app" : "demo-deploy"

                    echo "⏳ Checking rollout for deployment: ${DEPLOY_NAME} in namespace: ${env.BRANCH_NAME}"

                    try {
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
