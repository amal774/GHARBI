pipeline {
    agent any
    environment {
        NEXUS_URL = '127.0.0.1:8081'
        NEXUS_REPO = 'nexusproject'
        NEXUS_USER = 'admin'
        NEXUS_PASSWORD = 'esprit'
        DOCKER_IMAGE = 'amal311/imagedocker'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_TOKEN = credentials('sonar-token')
        NEXUS_VERSION = '1.0.0'
        NEXUS_PROTOCOL = 'http'
        NEXUS_REPOSITORY = 'nexusproject'
        NEXUS_CREDENTIAL_ID = 'nexus'
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
                echo "test"
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
                    // Uploading the artifact to Nexus
                    nexusArtifactUploader(
                        nexusVersion: '3', // Use Nexus 3
                        protocol: 'http',  // Ensure protocol is correct (http or https)
                        nexusUrl: "${NEXUS_URL}",  // Nexus URL
                        groupId: 'tn.esprit',  // Your project's groupId
                        version: '0.0.1-SNAPSHOT',  // The version you're deploying
                        repository: "${NEXUS_REPO}",  // Nexus repository for deployment
                        credentialsId: "${NEXUS_CREDENTIALS}",  // Credentials for Nexus access
                        artifacts: [
                            [artifactId: 'ExamThourayaS2', classifier: '', file: 'target/ExamThourayaS2-0.0.1-SNAPSHOT.jar', type: 'jar'],  // Adjust artifact details
                            [artifactId: 'ExamThourayaS2', classifier: '', file: 'target/ExamThourayaS2-0.0.1-SNAPSHOT.pom', type: 'pom']  // Ensure POM file is included
                        ]
                    )
                }
            }
        }
                }
            }
        }
    }
}

