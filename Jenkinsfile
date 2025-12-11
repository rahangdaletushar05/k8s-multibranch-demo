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
