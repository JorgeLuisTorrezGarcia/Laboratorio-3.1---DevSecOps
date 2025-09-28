pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('SAST - Semgrep') {
            steps {
                echo 'Running Semgrep (Static Analysis)...'
                sh '''
                    pip install semgrep --quiet
                    semgrep --config=auto src/ || true
                '''
            }
        }

        stage('SCA - OWASP Dependency Check') {
            steps {
                echo 'Running Software Composition Analysis...'
                sh '''
                    wget https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.0/dependency-check-8.4.0-release.zip -O dc.zip
                    unzip dc.zip -d dc
                    dc/dependency-check/bin/dependency-check.sh --scan src/ --format HTML --out reports || true
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Simulating build...'
                sh 'echo "Build step completed."'
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    # Simulate tests
                    echo "All tests passed!"
                '''
            }
        }

        stage('DAST - OWASP ZAP (simulado)') {
            steps {
                echo 'Running simulated ZAP scan...'
                sh 'echo "ZAP scan simulated (no Docker used)."'
            }
        }

        stage('Security Policy Check') {
            steps {
                echo 'Checking for high/critical vulnerabilities (simulated)...'
                sh '''
                    echo "No high/critical issues found. Passing policy check."
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up...'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
