#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "ibrahimelothmani98/blog-platform-backend:latest"
        FRONTEND_IMAGE = "ibrahimelothmani98/blog-platform-frontend:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: 'https://github.com/ibrahimelothmani/Blog-Platform-CloudOps.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh './mvnw clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t $BACKEND_IMAGE ./backend"
                    sh "docker build -t $FRONTEND_IMAGE ./frontend"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push $BACKEND_IMAGE"
                        sh "docker push $FRONTEND_IMAGE"
                    }
                }
            }
        }

        stage('Deploy with Docker') {
            steps {
                script {
                    // Stop old containers if running
                    sh 'docker rm -f backend || true'
                    sh 'docker rm -f frontend || true'

                    // Run new containers
                    sh "docker run -d --name backend -p 8080:8080 $BACKEND_IMAGE"
                    sh "docker run -d --name frontend -p 3000:80 $FRONTEND_IMAGE"
                }
            }
        }
    }
}
