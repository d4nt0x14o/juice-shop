pipeline {
    agent any

    tools {
        nodejs "NodeJS"              // Must be defined in Jenkins Global Tool Config
    }

    stages {
        stage('Checkout Juice Shop Source') {
            steps {
                deleteDir() // clean workspace
                git branch: 'master',
                    url: 'https://github.com/d4nt0x14o/juice-shop.git',
                    credentialsId: 'git-https-credentials'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Dependency Check') {
            steps {
                sh '''
                docker run --rm \
                  -v $(pwd):/src \
                  -v $(pwd)/odc-data:/report \
                  owasp/dependency-check:latest \
                  --scan /src --format "ALL" --out /report
                '''
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_SCANNER_HOME = tool 'SonarScanner'  // Must be configured in Jenkins
            }
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                        sh """
                        \$SONAR_SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectKey=juice-shop \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=\$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                sh '''
                docker run --rm \
                  -v $(pwd):/zap/wrk/:rw \
                  --network devsecops-net \
                  -t ghcr.io/zaproxy/zaproxy:stable \
                  zap-baseline.py -t http://juice-shop:3000 -r zap_report.html
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/*.html, **/*.xml, **/*.json', allowEmptyArchive: true
        }
    }
}
