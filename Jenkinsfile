pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh 'python3 -m venv env'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '. env/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Application') {
            steps {
                sh '. env/bin/activate && python3 main.py'
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
