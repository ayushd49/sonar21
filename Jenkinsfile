pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests & Coverage') {
            steps {
                echo 'Running tests with coverage...'
                sh '''
                    python3 -m pytest tests/ \
                        --cov=src \
                        --cov-report=xml:coverage.xml \
                        --cov-report=term-missing \
                        -v
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarScanner') {   
                    sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=sonar-demo21 \
                            -Dsonar.sources=src \
                            -Dsonar.tests=tests \
                            -Dsonar.python.coverage.reportPaths=coverage.xml
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate...'
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality Gate FAILED: ${qg.status}"
                        } else {
                            echo "Quality Gate PASSED: ${qg.status}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline passed! Code quality is good.'
        }
        failure {
            echo 'Pipeline failed! Check SonarQube report for issues.: ${FAILED_STAGE}'
        }
        always {
            echo 'Pipeline completed. Check SonarQube dashboard for detailed report.'
        }
    }
}
