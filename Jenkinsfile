pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
        jdk 'JDK17'
    }

    environment {
        // --- SONAR SETTINGS ---
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN    = credentials("sonar-token")

        // --- EMAIL SETTINGS ---
        TO_EMAIL       = "capoamine44@gmail.com"

        // --- DOCKER SETTINGS ---
        DOCKER_IMAGE   = "amineazizi/devops-project" 
        
        
        DOCKER_CREDS   = credentials('docker-hub-credentials')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/amine1azizi/devops.git'
            }
        }

        stage('Build & Test') {
            steps {
                // Compile the Java code
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube') {
            steps {
                sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=MyProject \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        // --- NEW STEP: Build Docker Image ---
        stage('Build & Push Docker') {
            steps {
                script {
                    // Log in to Docker Hub using the credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        
                        // Build the image
                        sh "docker build -t ${DOCKER_IMAGE}:latest ."
                        
                        // Push it to Docker Hub so Minikube can download it
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        // --- Deploy to Kubernetes ---
        stage('Deploy to K8s') {
            steps {
                // Create namespace if it doesn't exist (ignores error if it does)
                sh 'kubectl create namespace devops || true'

                // Deploy Database
                sh 'kubectl apply -f k8s/mysql-deployment.yaml -n devops'

                // Deploy Spring Boot App
                sh 'kubectl apply -f k8s/spring-deployment.yaml -n devops'

                // Force Kubernetes to download the new image
                sh 'kubectl rollout restart deployment/spring-app -n devops'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
            // Archive the jar
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
        }

        success {
            mail to: "${TO_EMAIL}",
                 subject: "SUCCESS: Pipeline ${currentBuild.fullDisplayName}",
                 body: "Great job! The build was successful.\nYour app is deployed on Kubernetes.\nCheck console: ${env.BUILD_URL}"
        }

        failure {
            mail to: "${TO_EMAIL}",
                 subject: "FAILURE: Pipeline ${currentBuild.fullDisplayName}",
                 body: "Something went wrong.\nCheck console: ${env.BUILD_URL}"
        }
    }
}
