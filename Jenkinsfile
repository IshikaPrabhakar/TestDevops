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
            # Create virtual environment
            python3 -m venv env
            . env/bin/activate

            # Use venv pip explicitly
            env/bin/pip install -r requirements.txt || true

            # Install snyk locally (no sudo issues)
            npm install snyk

            # Authenticate
            npx snyk auth $SNYK_TOKEN

            # Run scans
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
      . zap_env/bin/activate

      echo "📂 Checking files"
      pwd
      ls -la

      # Install dependencies if present
      if [ -f requirement.txt ]; then
        echo "📦 Installing dependencies"
        zap_env/bin/pip install -r requirement.txt
      else
        echo "⚠️ No requirement.txt found, skipping..."
      fi

      # Install required packages
      zap_env/bin/pip install python-owasp-zap-v2.4 setuptools uvicorn fastapi || true

      echo "🚀 Starting FastAPI app"
      nohup zap_env/bin/uvicorn main:app --host 0.0.0.0 --port 8000 > zap_app.log 2>&1 &
      APP_PID=$!

      echo "⏳ Waiting for app to be ready..."

      # Smart wait
      for i in {1..10}; do
        if curl -s http://127.0.0.1:8000 > /dev/null; then
          echo "✅ App is up!"
          break
        fi
        echo "Waiting..."
        sleep 5
      done

      # Get host IP
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

      echo "📄 Checking report file..."
      ls -la zap_report.html || echo "❌ Report not found"

      echo "🛑 Stopping app"
      kill $APP_PID || true
    '''

    publishHTML([
      allowMissing: true,   // prevents failure if report missing
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