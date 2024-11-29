pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'amal311/imagedocker'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_TOKEN = credentials('sonar-token')  // Replace with your Jenkins credential name for SonarQube
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/amal774/GHARBI.git'
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // SonarQube analysis with the authentication token
                sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=GHARBI_PROJECT \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                """
            }
        }

        stage("Docker Test") {
            steps {
                echo "Test Docker"
                sh "docker -v"
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh """
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push Docker image to DockerHub
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}

