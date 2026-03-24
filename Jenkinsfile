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
        withCredentials([string(credentialsId: 'Snyk_Ishika', variable: 'SNYK_TOKEN')]) {
          sh '''
            echo "🔐 Running Snyk Scan"

            python3 -m venv env

            # Install dependencies safely
            if [ -f requirements.txt ]; then
              env/bin/pip install -r requirements.txt
            else
              echo "⚠️ No requirements.txt found"
            fi

            npm install snyk

            npx snyk auth $SNYK_TOKEN

            npx snyk test --all-projects || true
            npx snyk monitor --all-projects || true
          '''
        }
      }
    }

    stage('DAST (ZAP Scan)') {
      steps {
        sh '''
          echo "🐍 Setting up environment"
          python3 -m venv zap_env

          echo "📂 Checking files"
          pwd
          ls -la

          # Install dependencies if present
          if [ -f requirements.txt ]; then
            zap_env/bin/pip install -r requirements.txt
          else
            echo "⚠️ No requirements.txt found"
          fi

          zap_env/bin/pip install python-owasp-zap-v2.4 setuptools uvicorn fastapi || true

          echo "🚀 Starting FastAPI app"
          nohup zap_env/bin/uvicorn main:app --host 0.0.0.0 --port 8000 > zap_app.log 2>&1 &
          APP_PID=$!

          echo "⏳ Waiting for app..."
          for i in {1..10}; do
            if curl -s http://127.0.0.1:8000 > /dev/null; then
              echo "✅ App is up!"
              break
            fi
            sleep 5
          done

          HOST_IP=$(hostname -I | awk '{print $1}')
          echo "🌐 Host IP: $HOST_IP"

          echo "🐳 Running ZAP scan"
          docker run --rm \
            --network host \
            -v $(pwd):/zap/wrk \
            owasp/zap2docker-stable \
            zap-baseline.py \
            -t http://$HOST_IP:8000 \
            -r /zap/wrk/zap_report.html || true

          echo "📄 Checking report"
          ls -la zap_report.html || echo "❌ Report not found"

          echo "🛑 Stopping app"
          kill $APP_PID || true
        '''

        publishHTML([
          allowMissing: true,
          alwaysLinkToLastBuild: true,
          keepAll: true,
          reportDir: '.',
          reportFiles: 'zap_report.html',
          reportName: 'ZAP DAST Report'
        ])

        archiveArtifacts artifacts: 'zap_report.html', fingerprint: true, allowEmptyArchive: true
      }
    }

    stage('Build Package') {
      steps {
        sh '''
          echo "📦 Setting up build environment"

          python3 -m venv env

          env/bin/pip install --upgrade pip
          env/bin/pip install build

          echo "🚀 Building package"
          env/bin/python -m build || echo "⚠️ Build failed (missing pyproject.toml/setup.py)"

          echo "📂 Checking dist folder"
          ls -la dist || true
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