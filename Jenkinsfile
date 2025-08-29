pipeline {
    agent any

    tools {
        nodejs "NodeJS"        // configure NodeJS in Jenkins -> Global Tool Configuration
    }

    stages {
        stage('Checkout Juice Shop Source') {
            steps {
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
                  owasp/dependency-check \
                  --scan /src --format "ALL" --out /report
                '''
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_SCANNER_HOME = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh '''
                    $SONAR_SCANNER_HOME/bin/sonar-scanner \
                      -Dsonar.projectKey=juice-shop \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                sh '''
                docker run --rm \
                  -v $(pwd):/zap/wrk/:rw \
                  -t ghcr.io/zaproxy/zaproxy:stable \
                  zap-baseline.py -t http://host.docker.internal:3000 -r zap_report.html
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
