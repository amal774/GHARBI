pipeline {
    agent any
    environment {
        NEXUS_URL = '127.0.0.1:8081'  // Nexus URL
        NEXUS_REPO = 'nexusproject'  // Nexus repository name
        NEXUS_USER = 'admin'  // Nexus username
        NEXUS_PASSWORD = 'esprit'  // Nexus password (not recommended to hard-code in Jenkinsfile, use credentials)
        DOCKER_IMAGE = 'amal311/imagedocker'  // Docker image name
        DOCKER_CREDENTIALS = 'dockerhub-credentials'  // Docker credentials stored in Jenkins
        SONARQUBE_URL = 'http://localhost:9000'  // SonarQube URL
        SONARQUBE_TOKEN = credentials('sonar-token')  // SonarQube token credentials
        NEXUS_VERSION = '0.0.1'  // Version for Nexus upload
        NEXUS_PROTOCOL = 'http'  // Nexus protocol (http or https)
        NEXUS_REPOSITORY = 'nexusproject'  // Nexus repository for deployment
        NEXUS_CREDENTIALS = 'nexus'  // Nexus credentials stored in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/amal774/GHARBI.git'
            }
        }

        stage('Build Project4') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh "mvn sonar:sonar -Dsonar.projectKey=GHARBI_PROJECT -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.login=${SONARQUBE_TOKEN}"    
            }
        }

        stage("DOCKER") {
            steps {
                echo "Checking Docker version"
                sh "docker -v"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Upload the artifact to Nexus
                    nexusArtifactUploader(
                        nexusVersion: '3', // Use Nexus 3
                        protocol: "${NEXUS_PROTOCOL}",  // Nexus protocol (http/https)
                        nexusUrl: "${NEXUS_URL}",  // Nexus URL
                        groupId: 'tn.esprit',  // Your project's groupId
                        version: "${NEXUS_VERSION}",  // Version you're deploying
                        repository: "${NEXUS_REPOSITORY}",  // Nexus repository for deployment
                        credentialsId: "${NEXUS_CREDENTIALS}",  // Credentials for Nexus access
                        artifacts: [
                            [artifactId: 'ExamThourayaS2', classifier: '', file: 'target/ExamThourayaS2-${NEXUS_VERSION}-SNAPSHOT.jar', type: 'jar'],  // Adjust artifact details
                            [artifactId: 'ExamThourayaS2', classifier: '', file: 'target/ExamThourayaS2-${NEXUS_VERSION}-SNAPSHOT.pom', type: 'pom']  // Ensure POM file is included
                        ]
                    )
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed. Please check the logs for errors.'
        }
    }
}

