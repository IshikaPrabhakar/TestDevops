pipeline {
  agent any

  environment {
    SONAR_URL = 'http://ec2-13-202-47-19.ap-south-1.compute.amazonaws.com:15998/'
    SONAR_PROJECT = 'Devops_Project'
    PROJECT_LANG = 'python'
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            credentialsId: 'github-pat',
            url: 'https://github.com/IshikaPrabhakar/TestDevops.git'
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh """
          sonar-scanner \
          -Dsonar.projectKey=Devops_Project \
	      -Dsonar.projectName=Devops_Project \
          -Dsonar.sources=. \
          -Dsonar.host.url=${SONAR_URL}
          """
        }
      }
    }

    stage('Snyk Scan') {
      steps {
        withCredentials([string(credentialsId: 'snyk-token01', variable: 'SNYK_TOKEN')]) {
          sh '''
            python3 -m venv env
            . env/bin/activate
            pip install -r requirements.txt || true

            npm install -g snyk
            npx snyk auth ${SNYK_TOKEN}

            snyk test --file=requirements.txt --package-manager=pip --severity-threshold=high || true
            snyk monitor --file=requirements.txt --package-manager=pip
          '''
        }
      }
    }

    stage('DAST (ZAP Scan)') {
      steps {
        sh '''
          echo "🐍 Setting up environment"
          python3 -m venv zap_env
          . zap_env/bin/activate
          pip install -r requirements.txt
          pip install python-owasp-zap-v2.4 setuptools

          echo "🚀 Starting FastAPI app"
          nohup zap_env/bin/uvicorn main:app --host 0.0.0.0 --port 8000 > zap_app.log 2>&1 &
          APP_PID=$!

          echo "⏳ Waiting for app..."
          sleep 15

          # Get machine IP (IMPORTANT FIX)
          HOST_IP=$(hostname -I | awk '{print $1}')
          echo "Host IP: $HOST_IP"

          echo "🐳 Running ZAP scan via Docker"
          docker run --rm \
            --network host \
            -v $(pwd):/zap/wrk \
            ghcr.io/zaproxy/zaproxy:weekly \
            zap-baseline.py \
            -t http://$HOST_IP:8000 \
            -r zap_report.html

          echo "🛑 Stopping app"
          kill $APP_PID || true
        '''

        publishHTML([
          allowMissing: false,
          alwaysLinkToLastBuild: true,
          keepAll: true,
          reportDir: '.',
          reportFiles: 'zap_report.html',
          reportName: 'ZAP DAST Report'
        ])

        archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
      }
    }

    stage('Build Package') {
      steps {
        sh '''
          python3 -m venv env
          . env/bin/activate
          pip install build
          python -m build
        '''
      }
    }

  }

  post {
    success {
      echo '✅ Pipeline executed successfully'
    }
    failure {
      echo '❌ Pipeline failed'
    }
  }
}