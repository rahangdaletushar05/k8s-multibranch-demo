@Library('my-shared-lib') _

pipeline {
    agent any

    environment {
        DOCKER_USER = "tusharrahangdale"
        IMAGE_TAG = "${env.BRANCH_NAME}".replaceAll('/', '-')
    }

    stages {

        /* ------------------ 1. Checkout ------------------ */
        stage("Checkout") {
            steps {
                echo "Running Branch: ${env.BRANCH_NAME}"
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
                    sh """
                    echo \$PASS | docker login -u \$USER --password-stdin
                    """
                }
