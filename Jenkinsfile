pipeline {
    agent any
    tools {
        nodejs 'node20.18'
    }

    environment {
        NEXUS_URL = 'http://13.232.231.44:8082/'    // URL to the Nexus docker repo
        NEXUS_REPO = 'docker-releases'         // Nexus docker repo name
        DOCKER_USERNAME = 'admin'                  // username of Nexus repo
        DOCKER_PASSWORD = 'vivek2003@'               // password of Nexus repo
        IMAGE_NAME = 'react'                       // name of image
        SONARQUBE_SERVER = 'Sonar'      // Jenkins SonarQube server name
        DOCKERHUB_NAME = 'vivekkrishnab' // Docker Hub username
        DOCKERHUB_REPO = 'docker-reactjs'            // Docker Hub repo name
        DOCKERHUB_PASSWORD = 'vivek2003@'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vivek-66/docker-reactjs.git', credentialsId: 'git-hook-token'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                rm -rf node_modules package-lock.json
                npm install
                npm cache clean --force
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=docker-react \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://13.232.231.44/:9000
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push Image to Nexus') {
            steps {
                sh '''
                echo "$DOCKER_PASSWORD" | docker login $NEXUS_URL -u "$DOCKER_USERNAME" --password-stdin
                docker tag $IMAGE_NAME:latest $NEXUS_URL/$NEXUS_REPO/$IMAGE_NAME:latest
                docker push $NEXUS_URL/$NEXUS_REPO/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_NAME" --password-stdin
                docker tag $IMAGE_NAME:latest $DOCKERHUB_NAME/$DOCKERHUB_REPO:latest
                docker push $DOCKERHUB_NAME/$DOCKERHUB_REPO:latest
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker compose down || true
                echo "version: '3'
                services:
                  react-app:
                    image: $DOCKERHUB_NAME/$DOCKERHUB_REPO:latest
                    ports:
                      - '3000:80'
                    restart: always" > docker-compose.yml
                docker compose up -d
                '''
            }
        }
    }
}

