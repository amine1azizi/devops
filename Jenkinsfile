pipeline {
    agent any

    // ADDED: From your new file (runs every 5 mins)
    triggers {
        cron('H/5 * * * *')
    }

    // ADDED: From your new file (stops build if stuck for 1 hour)
    options {
        timeout(time: 1, unit: 'HOURS')
    }

    tools {
        // KEPT OLD: We use the names that worked for you ('MAVEN_HOME', 'JDK17')
        // If you change these to 'maven3', your build might break if Jenkins doesn't know that name.
        maven 'MAVEN_HOME'
        jdk 'JDK17'
    }

    environment {
        // KEPT OLD: Sonar credentials
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN = credentials("sonar-token")
        
        // ADDED: From your new file
        APP_ENV = "DEV"
    }

    stages {

        stage('Code Checkout') {
            steps {
                echo "=====Checking out code from Git====="
                // UPDATED: usage of 'checkout scm' is better than hardcoding the URL.
                // This allows the Jenkins Job Configuration to decide which Repo to use.
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "=====Building Spring Boot application====="
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                // KEPT OLD: It is good practice to keep your tests!
                sh 'mvn test'
            }
        }

        stage('Sonar') {
            steps {
                // KEPT OLD: SonarQube analysis
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

            // KEPT OLD: JUnit reports
            script {
                if (fileExists('target/surefire-reports')) {
                    junit 'target/surefire-reports/*.xml'
                }
            }

            // MERGED: Archive artifacts with 'fingerprint: true' from new file
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false, fingerprint: true
        }
        
        // ADDED: From your new file
        success {
            echo "=====pipeline executed successfully ====="
        }
        failure {
            echo "======pipeline execution failed======"
        }
    }
}
