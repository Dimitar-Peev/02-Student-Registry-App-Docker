pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'mctrix87'
        APP_NAME        = 'student-registry-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        IMAGE_FULL      = "${DOCKER_USERNAME}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout the repository') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %IMAGE_FULL% ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    """
                }
            }
        }

        stage('Docker Push') {
            steps {
                bat """
                docker push %IMAGE_FULL%
                docker logout
                """
            }
        }

        stage('Deploy (CD)') {
            when {
                branch 'main'
            }
            steps {
                bat """
                docker pull %IMAGE_FULL%
                docker-compose -f docker-compose.yml up -d --force-recreate --remove-orphans
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo "✅ Deployment successful: ${IMAGE_FULL}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        cleanup {
            cleanWs(deleteDirs: true, notFailBuild: true)
        }
    }
}
