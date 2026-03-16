pipeline {
agent any

```
environment {
    SONAR_SCANNER_HOME = tool 'SonarScanner'
    SNYK_TOKEN = credentials('snyk-token')
}

stages {

    stage('Checkout Code') {
        steps {
            git url: 'https://github.com/IshikaPrabhakar/TestDevops.git', branch: 'main'
        }
    }

    stage('Create Virtual Environment') {
        steps {
            sh '''
            rm -rf env || true
            python3 -m venv env
            '''
        }
    }

    stage('Install Dependencies') {
        steps {
            sh '''
            . env/bin/activate
            pip install --upgrade pip
            pip install -r requirements.txt
            pip install python-owasp-zap-v2.4 setuptools
            '''
        }
    }

    stage('Snyk Security Scan') {
        steps {
            sh '''
            npm install -g snyk
            snyk auth $SNYK_TOKEN
            snyk test || true
            '''
        }
    }

    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('sonarqube-server') {
                sh '''
                $SONAR_SCANNER_HOME/bin/sonar-scanner
                '''
            }
        }
    }

    stage('Run Application') {
        steps {
            sh '''
            . env/bin/activate
            nohup uvicorn main:app --host 0.0.0.0 --port 8000 > app.log 2>&1 &
            sleep 15
            '''
        }
    }

    stage('OWASP ZAP Security Scan') {
        steps {
            sh '''
            . env/bin/activate
            python zap-baseline.py -t http://localhost:8000 -r zap_report.html
            '''
        }
    }
}

post {
    always {
        archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
    }

    success {
        echo 'Pipeline executed successfully'
    }

    failure {
        echo 'Pipeline failed'
    }
}
```

}
