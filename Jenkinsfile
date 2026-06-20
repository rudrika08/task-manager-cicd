pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP_NAME     = 'task-manager'
        DOCKER_USER  = 'rudrika83'
        DOCKER_IMAGE = "${DOCKER_USER}/${APP_NAME}"
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -B clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw -B test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DUSER',
                    passwordVariable: 'DPASS'
                )]) {
                    sh '''
                        docker build -t "${DOCKER_IMAGE}:${BUILD_NUMBER}" .
                        printf '%s\n' "$DPASS" | docker login -u "$DUSER" --password-stdin
                        docker push "${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        docker tag "${DOCKER_IMAGE}:${BUILD_NUMBER}" "${DOCKER_IMAGE}:latest"
                        docker push "${DOCKER_IMAGE}:latest"
                    '''
                }
            }
            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['prod-server-ssh']) {
                    sh 'ansible-playbook ansible/deploy.yml -i ansible/inventory -e "image_tag=${BUILD_NUMBER}" -e "docker_user=${DOCKER_USER}"'
                }
            }
        }
    }

    post {
    failure {
        echo "❌ Pipeline FAILED for ${APP_NAME}:${BUILD_NUMBER} — check stage logs above"

        // 📧 Email Notification (Requires SMTP Configuration in Jenkins)
        // emailext(
        //     to: 'rudrika.812@gmail.com',
        //     subject: "❌ Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
        //     body: "The pipeline failed. Please check the logs here: ${env.BUILD_URL}",
        //     mimeType: 'text/plain'
        // )

        // 🔔 Slack Notification (Requires Slack Notification Plugin)
        // slackSend(
        //     channel: '#alerts',
        //     color: 'danger',
        //     message: "❌ Pipeline FAILED: ${env.JOB_NAME} [${env.BUILD_NUMBER}]\nView Logs: ${env.BUILD_URL}"
        // )
    }
    success {
        echo "✅ Successfully deployed ${APP_NAME}:${BUILD_NUMBER} to production"

        // 📧 Email Notification (Requires SMTP Configuration in Jenkins)
        // emailext(
        //     to: 'rudrika.812@gmail.com',
        //     subject: "✅ Build Successful: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
        //     body: "The pipeline deployed successfully. View it here: ${env.BUILD_URL}",
        //     mimeType: 'text/plain'
        // )

        // 🔔 Slack Notification (Requires Slack Notification Plugin)
        // slackSend(
        //     channel: '#alerts',
        //     color: 'good',
        //     message: "✅ Successfully deployed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]\nView: ${env.BUILD_URL}"
        // )
    }
    aborted {
        echo "⚠️ Pipeline aborted for ${APP_NAME}:${BUILD_NUMBER}"
    }
}
}
