pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        S3_BUCKET = "mongobackup0123"
        MONGO_SECRET_NAME = "mongo/creds"
        MONGO_HOST = "mdb.spanllc.com"
        MONGO_PORT = "27017"
    }

    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    try {
                        // Run Gitleaks
                        sh 'gitleaks detect --source . --report-format json --report-path gitleaks-report.json'
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Gitleaks found issues!'
                    }
                }
            }
        }
        
