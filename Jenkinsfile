@Library('my-shared-lib') _

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
        IMAGE_TAG = "${env.BRANCH_NAME}".replaceAll('/', '-')
    }

    stages {

        stage("Checkout") {
            steps {
                echo "🔹 Running Branch → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage("Docker Login") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds-2',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                sh """
                docker build -t ${DOCKER_USER}/demo-app:${IMAGE_TAG} .
                docker push ${DOCKER_USER}/demo-app:${IMAGE_TAG}
                """
            }
        }

        stage("Deploy To Kubernetes") {
            steps {
                k8sDeploy(
                    image: "${DOCKER_USER}/demo-app:${IMAGE_TAG}",
                    namespace: env.BRANCH_NAME,
                    deployFile: "k8s/deployment.yaml",
                    credential: "k8s-config"
                )
            }
        }

        stage("Verify Rollout Status") {
            steps {
                script {

                    def DEPLOY_NAME = ""

                    if (env.BRANCH_NAME == "main") {
                        DEPLOY_NAME = "k8s-app"
                    } else if (env.BRANCH_NAME == "prod") {
                        DEPLOY_NAME = "prod-app"
                    } else {
                        DEPLOY_NAME = "demo-deploy"
                    }

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
