pipeline {
    agent any

    environment {
        APP_NAME     = 'task-manager'
        DOCKER_USER  = 'rudrika83'
        DOCKER_IMAGE = "${DOCKER_USER}/${APP_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
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
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                        echo $DPASS | docker login -u $DUSER --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "ansible-playbook ansible/deploy.yml -i ansible/inventory -e image_tag=${BUILD_NUMBER}"
            }
        }
    }

    post {
        failure {
            echo 'Build failed — check logs'
        }
        success {
            echo "Deployed ${APP_NAME}:${BUILD_NUMBER}"
        }
    }
}
