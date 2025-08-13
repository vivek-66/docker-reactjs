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
                git branch: 'master', url: 'https://github.com/vivek-66/docker-reactjs.git', credentialsId: 'github-creds'
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
                        -Dsonar.host.url=http://13.232.231.44:9000
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $vivekkrishnab/docker-react .'
            }
        }

        stage('Push Image to Nexus') {
            steps {
                sh '''
                echo "vivek2003@" | docker login http://13.232.231.44:8082 -u "admin" --password-stdin
                docker tag vivekkrishnab/docker-react:latest react/docker-release/docker-react:latest
                docker push http://13.232.231.44:8082/docker-release/docker-react:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                echo "vivek2003@" | docker login -u "vivekkrishnab" --password-stdin
                docker tag vivekkrishnab/docker-react/vivekkrishnab/docker-react:latest
                docker push vivekkrishnab/docker-react:latest
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
                    image: vivekkrishnab/docker-react:latest
                    ports:
                      - '3001:80'
                    restart: always" > docker-compose.yml
                docker compose up -d
                '''
            }
        }
    }
}

