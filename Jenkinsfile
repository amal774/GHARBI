pipeline {
    agent any
    environment {
        NEXUS_URL = 'http://127.0.0.1:8081'
        NEXUS_REPO = 'nexusproject'
        NEXUS_USER = 'admin'
        NEXUS_PASSWORD = 'esprit'
        DOCKER_IMAGE = 'amal311/imagedocker'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_TOKEN = credentials('sonar-token')  // Remplace par le nom de ton credential Jenkins
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
                // Correction ici : Utilisation de la bonne syntaxe avec des variables d'environnement.
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
        
        stage('PUSH TO NEXUS') {
        	steps {
        		script {
                    		def file = 'target/ExamThourayaS2.jar'
                    		def groupId = 'com.example'
                    		def artifactId = 'my-project'
                    		def version = '1.0.0'
                    		sh """curl -u $NEXUS_USER:$NEXUS_PASSWORD --upload-file $file \ $NEXUS_URL/repository/$NEXUS_REPO/$groupId/$artifactId/$version/$artifactId-$version.jar"""
                	}
                }
        }
        }
    }
}

