pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
        jdk 'JDK17'
    }

    environment {
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN = credentials("sonar-token")
    }

    stages {

        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/amine1azizi/mavenProject.git'
            }
        }

        stage('Build') {
            steps {
                // Compile + package jar, skip tests for now
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                // Run tests separately
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

            // Publish test reports if any
            script {
                if (fileExists('target/surefire-reports')) {
                    junit 'target/surefire-reports/*.xml'
                } else {
                    echo "No test reports found."
                }
            }

            // Archive the jar artifact
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
        }
    }
}
