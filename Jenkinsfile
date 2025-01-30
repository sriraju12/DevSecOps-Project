pipeline {
    agent any

    tools {
        jdk 'Jdk17'
        nodejs 'node16'
    }

    environment {
        DOCKER_IMAGE = "sriraju12/netflix-app:${BUILD_NUMBER}"
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/sriraju12/DevSecOps-Project.git'
            }
        }

         stage('sonarqube analysis') {
          steps {
            script {
                withSonarQubeEnv('sonar-server'){
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
          }
        }

        stage('install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-CHECK'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Scan File System') {
            steps {
                sh "/opt/homebrew/bin/trivy fs --format table -o trivy-fs-reports.html ."
            }
        }

        stage('Build and Push Docker Image') {
          environment {
             REGISTRY_CREDENTIALS = credentials('dockerhub-token')
             TMDB_TOKEN_KEY = credentials('TMDB_Token')
            }
          steps {
            script {
              sh 'docker context use default'  
              sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_TOKEN_KEY} -t ${DOCKER_IMAGE} ."
              def dockerImage = docker.image("${DOCKER_IMAGE}")
              docker.withRegistry('https://index.docker.io/v1/', "dockerhub-token") {
                  dockerImage.push()
                }
            }
          }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE}"
            }
        }
    }
}
