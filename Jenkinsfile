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
        DOCKERHUB_NAME = 'vivekkishnab' // Docker Hub username
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
                sh 'CI=false npm test -- --watchAll=false || true'
          }
     }

   stage('SonarQube Analysis') {
    steps {
        script {
            // Get the path to the SonarQube Scanner tool installation
            def scannerHome = tool 'sonarscanner'  // Must match the name in Jenkins Global Tool Config

            // Inject SonarQube environment variables
            withSonarQubeEnv('sonar') {            // Must match the name in SonarQube servers config
                sh """
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=docker-react \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://13.232.231.44:9000
                """
            }
        }
    }
}

       stage('Build Docker Image') {
    steps {
        sh 'docker build -t vivekkishnab/docker-react .'
    }
}


        stage('Push Image to Nexus') {
    steps {
        sh '''
        echo "vivek2003@" | docker login 13.232.231.44:8082 -u "admin" --password-stdin
        docker tag vivekkishnab/docker-react:latest 13.232.231.44:8082/docker-release/docker-react:latest
        docker push 13.232.231.44:8082/docker-release/docker-react:latest
        '''
    }
}



       stage('Push Image to Docker Hub') {
    steps {
        sh '''
        echo "vivek2003@" | docker login -u "vivekkishnab" --password-stdin
        docker tag vivekkishnab/docker-react:latest vivekkishnab/docker-react:latest
        docker push vivekkishnab/docker-react:latest
        '''
    }
}

          stage('Deploy Application') {
          steps {
              sh '''
              docker compose down || true
              cat <<EOF > docker-compose.yml
version: '3'
services:
  react-app:
    image: vivekkrishnab/docker-react:latest
    ports:
      - "3001:80"
    restart: always
EOF
              docker compose up -d
              '''
          }
      }
    }  // <-- closes stages
}      // <-- closes pipeline

