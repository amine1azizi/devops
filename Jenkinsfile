pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
        jdk 'JDK17'
    }

    environment {
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN = credentials("sonar-token")
        TO_EMAIL = "capoamine44@gmail.com" 
    }

    stages {
        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/amine1azizi/devops.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar') {
            steps {
                sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=MyProject \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
            
            // JUnit Reports
            script {
                if (fileExists('target/surefire-reports')) {
                    junit 'target/surefire-reports/*.xml'
                }
            }
            
            // Archive Jar
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
        }

        success {
            mail to: "${TO_EMAIL}",
                 subject: "SUCCESS: Pipeline ${currentBuild.fullDisplayName}",
                 body: "Great job! The build was successful.\nCheck console: ${env.BUILD_URL}"
        }

        failure {
            mail to: "${TO_EMAIL}",
                 subject: "FAILURE: Pipeline ${currentBuild.fullDisplayName}",
                 body: "Something went wrong.\nCheck console: ${env.BUILD_URL}"
        }
    }
}
