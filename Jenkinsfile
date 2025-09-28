pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = "devsecops-labs/app:latest"
    STAGING_URL = "http://localhost:3000"
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'ls -la'
      }
    }

    stage('SAST - Semgrep') {
      steps {
        script {
          docker.image('returntocorp/semgrep:latest').inside {
            sh '''
              semgrep --config=auto --json --output semgrep-results.json src || true
              cat semgrep-results.json || true
            '''
          }
        }
        archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
      }
    }

    stage('SCA - Dependency Check') {
      steps {
        script {
          docker.image('owasp/dependency-check:latest').inside {
            sh '''
              mkdir -p dependency-check-reports
              dependency-check --project "devsecops-labs" --scan . --format JSON --out dependency-check-reports || true
            '''
          }
        }
        archiveArtifacts artifacts: 'dependency-check-reports/**', allowEmptyArchive: true
      }
    }

    stage('Build') {
      steps {
        sh '''
          cd src
          npm install --no-audit --no-fund
          if [ -f package.json ]; then
            if npm test --silent; then echo "Tests OK"; else echo "Tests failed (continue)"; fi
          fi
        '''
      }
    }

    stage('Docker Build & Trivy Scan') {
      steps {
        sh '''
          docker build -t ${DOCKER_IMAGE_NAME} -f Dockerfile .
          mkdir -p trivy-reports
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format json --output trivy-reports/trivy-report.json ${DOCKER_IMAGE_NAME} || true
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --severity HIGH,CRITICAL ${DOCKER_IMAGE_NAME} || true
        '''
        archiveArtifacts artifacts: 'trivy-reports/**', allowEmptyArchive: true
      }
    }

    stage('Deploy to Staging') {
      steps {
        sh '''
          docker-compose -f docker-compose.yml down || true
          docker-compose -f docker-compose.yml up -d --build
          sleep 8
        '''
      }
    }

    stage('DAST - ZAP Scan') {
      steps {
        sh '''
          mkdir -p zap-reports
          docker run --rm --network host owasp/zap2docker-stable zap-baseline.py -t ${STAGING_URL} -r zap-reports/zap-report.html || true
        '''
        archiveArtifacts artifacts: 'zap-reports/**', allowEmptyArchive: true
      }
    }

    stage('Policy Check - Fail on HIGH/CRITICAL CVEs') {
      steps {
        sh '''
          chmod +x scripts/scan_trivy_fail.sh
          ./scripts/scan_trivy_fail.sh ${DOCKER_IMAGE_NAME} || exit_code=$?
          if [ "${exit_code:-0}" -eq 2 ]; then
              echo "Failing pipeline due to HIGH/CRITICAL vulnerabilities detected by Trivy."
              exit 1
          fi
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Collecting artifacts..."
    }
    failure {
      echo "Pipeline failed!"
    }
  }
}
