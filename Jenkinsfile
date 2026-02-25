pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/IshikaPrabhakar/TestDevops.git', branch: 'main'
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh 'rm -rf env || true'
                sh 'python3 -m venv env'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '. env/bin/activate && pip install -r requirement.txt'
            }
        }

        stage('Run Application') {
            steps {
                sh '. env/bin/activate && nohup uvicorn main:app --host 0.0.0.0 --port 8000 > app.log 2>&1 &'            
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}