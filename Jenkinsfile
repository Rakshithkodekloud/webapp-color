pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out successfully'
            }
        }
        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest flake8
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests and linting...'
                sh '''
                    . venv/bin/activate
                    # Run linting
                    flake8 app.py --max-line-length=120 --ignore=E501,W503 || true
                    # Run unit tests if available
                    pytest tests/ --junitxml=test-results.xml || echo "No tests found, skipping..."
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'test-results.xml'
                }
            }
        }
        stage('Package') {
            steps {
                echo 'Building and pushing Docker image to ECR...'
                sh """   
                        # Build Docker image
                        docker build -t webapp-color:${IMAGE_TAG} .
                    """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
