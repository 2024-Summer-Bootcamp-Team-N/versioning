pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose-deploy.yml'
        DEPLOY_SERVER="mocodoco@43.200.97.88"  // 배포가 진행되는 서버 주소 입니다.
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main', url: 'https://github.com/2024-Summer-Bootcamp-Team-N/versioning.git'
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker --version"
                    sh "docker compose --version"
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                script {
                    sshagent(['deploy-server-access']) {
                        sh """
                        scp -o StrictHostKeyChecking=no ${DOCKER_COMPOSE_FILE} ${DEPLOY_SERVER}:~/${DOCKER_COMPOSE_FILE}
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        cd ~ &&
                        ls -al &&
                        docker compose -f ${DOCKER_COMPOSE_FILE} pull &&
                        docker compose -f ${DOCKER_COMPOSE_FILE} up -d'
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                            [pattern: '.propsfile', type: 'EXCLUDE']])
        }
        success {
            echo 'Build and deployment successful!'
            slackSend message: "Service deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        failure {
            echo 'Build or deployment failed.'
            slackSend failOnError: true, message: "Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
    }
}
