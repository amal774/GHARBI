pipeline {
    agent any
    environment {
        NEXUS_URL = '127.0.0.1:8081'  // URL de Nexus
        NEXUS_REPO = 'nexusproject'  // Nom du dépôt Nexus
        NEXUS_USER = 'admin'  // Nom d'utilisateur Nexus
        NEXUS_PASSWORD = 'esprit'  // Mot de passe Nexus (à éviter en clair, utilisez plutôt des credentials Jenkins)
        DOCKER_IMAGE = 'amal311/imagedocker'  // Nom de l'image Docker
        DOCKER_CREDENTIALS = 'dockerhub-credentials'  // Identifiants Docker enregistrés dans Jenkins
        SONARQUBE_URL = 'http://localhost:9000'  // URL de SonarQube
        SONARQUBE_TOKEN = credentials('sonar-token')  // Token SonarQube stocké dans Jenkins
        NEXUS_VERSION = '0.0.1'  // Version à déployer dans Nexus
        NEXUS_PROTOCOL = 'http'  // Protocole Nexus (http ou https)
        NEXUS_REPOSITORY = 'nexusproject'  // Dépôt Nexus pour déploiement
        NEXUS_CREDENTIALS = 'nexus'  // Identifiants Nexus enregistrés dans Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupère le code depuis GitHub
                git 'https://github.com/amal774/GHARBI.git'
            }
        }

        stage('Build Project') {
            steps {
                // Compile et package le projet avec Maven
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Lancer l'analyse SonarQube pour la qualité du code
                sh "mvn sonar:sonar -Dsonar.projectKey=GHARBI_PROJECT -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.login=${SONARQUBE_TOKEN}"
            }
        }

        stage("Check Docker") {
            steps {
                // Vérifie la version de Docker installée
                echo "Checking Docker version"
                sh "docker -v"
            }
        }

        stage('Build Docker Image') {
            steps {
                // Crée l'image Docker en utilisant le Dockerfile du projet
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push l'image Docker dans le repository Docker Hub
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                // Déploiement des artefacts dans Nexus avec Maven
                script {
                    // Utilisation de `mvn deploy` pour envoyer l'artefact vers Nexus
                    sh """
                        mvn deploy:deploy-file \
                        -DgroupId=tn.esprit \
                        -DartifactId=ExamThourayaS2 \
                        -Dversion=${NEXUS_VERSION} \
                        -Dpackaging=jar \
                        -Dfile=target/ExamThourayaS2-0.0.1-SNAPSHOT.jar \
                        -DrepositoryId=nexus-repository \
                        -Durl=${NEXUS_PROTOCOL}://${NEXUS_URL}/repository/${NEXUS_REPOSITORY} \
                        -Dusername=${NEXUS_USER} \
                        -Dpassword=${NEXUS_PASSWORD}
                    """
                }
            }
        }
    }

    post {
        success {
            // Si tout se passe bien
            echo 'Build and deployment successful!'
        }
        failure {
            // Si une étape échoue
            echo 'Build or deployment failed. Please check the logs for errors.'
        }
    }
}

