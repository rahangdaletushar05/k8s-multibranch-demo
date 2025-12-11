@Library('my-shared-lib') _

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"

        // Docker tag safe for slashes like feature/login
        IMAGE_TAG = "${env.BRANCH_NAME}".replaceAll('/', '-')
    }

    stages {

        /* ------------------ 1. Checkout ------------------ */
        stage("Checkout") {
            steps {
                echo "🔹 Running Branch → ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        /* ------------------ 2. Docker Login ------------------ */
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

        /* ------------------ 3. Docker Build & Push ------------------ */
        stage("Docker Build & Push") {
            steps {
                sh """
                docker build -t ${DOCKER_USER}/demo-app:${IMAGE_TAG} .
                docker push ${DOCKER_USER}/demo-app:${IMAGE_TAG}
                """
            }
        }

        /* ------------------ 4. Deploy To Kubernetes ------------------ */
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

        /* ------------------ 5. Verify Rollout ------------------ */
        stage("Verify Rollout Status") {
            steps {
                script {

                    // Dynamic Deployment Naming Logic
                    def DEPLOY_NAME = ""

                    if (env.BRANCH_NAME == "main") {
                        DEPLOY_NAME = "k8s-app"
                    } else if (env.BRANCH_NAME == "prod") {
                        DEPLOY_NAME = "prod-app"
                    } else if (env.BRANCH_NAME == "stage") {
                        DEPLOY_NAME = "stage-app"
                    } else if (env.BRANCH_NAME == "dev") {
                        DEPLOY_NAME = "dev-app"
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
                        error("Rollback Triggered — Deployment Faile
