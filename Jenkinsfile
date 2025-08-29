pipeline {
    agent any
    tools {
        maven 'Maven3'   // Configure Maven under Jenkins Tools
        jdk 'jdk11'      // Configure JDK 11
    }
    environment {
        SONARQUBE = 'SonarQubeServer' // Configure in Jenkins global settings
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('SAST - SonarQube (Fortify later)') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('SCA - Dependency Check (Sonatype later)') {
            steps {
                sh 'dependency-check.sh --scan . --format XML --out reports/'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh 'docker run -d --name juice-shop-test -p 3001:3000 bkimminich/juice-shop'
            }
        }
        stage('DAST - OWASP ZAP (Qualys later)') {
            steps {
                sh 'docker run --rm owasp/zap2docker-stable zap-baseline.py -t http://host.docker.internal:3001 -r zap_report.html'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/reports/*, zap_report.html', fingerprint: true
        }
        failure {
            mail to: 'dteo@starhub.com',
                 subject: "Pipeline Failed",
                 body: "Security pipeline failed. Check Jenkins logs."
        }
    }
}
