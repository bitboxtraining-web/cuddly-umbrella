pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven-3'
    }

    environment {
        APP_NAME = "demo-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Approval') {
            steps {
                script {
                    input message: "Lancer le build Maven ?", ok: "Oui"
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn -B clean package"
            }
            post {
                success {
                    archiveArtifacts artifacts: "target/*.jar", fingerprint: true
                }
            }
        }

        stage('Tests') {
            steps {
                sh "mvn -B test"
            }
        }

        stage('Quality Checks (parallel)') {
            parallel {
                stage('Lint') {
                    steps { echo "Checkstyle…" }
                }
                stage('Security') {
                    steps { echo "Scan SAST…" }
                }
            }
        }

        stage('Docker Build + Run') {
            steps {
                sh '''
                    docker build -t demo-app:latest .
                    docker stop demo-container || true
                    docker rm demo-container || true
                    docker run -d --name demo-container -p 8081:8080 demo-app:latest
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline terminé avec Jenkins 2.528.2"
        }
    }
}
