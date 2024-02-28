pipeline {
    agent { label 'newagent1' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/claudedevops/register-app.git']])
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test  Application') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage('Build & Push Docker Image') {
            environment {
                DOCKER_IMAGE = 'claudeke/register-app-pipeline'
                DOCKER_TAG = 'latest'
                DOCKER_REGISTRY = 'docker.io'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential') {
                        sh "docker build -t ${DOCKER_IMAGE}:$DOCKER_TAG ."
                        sh "docker push ${DOCKER_IMAGE}:$DOCKER_TAG"
                    }
                }
            }
        }
    }
}
