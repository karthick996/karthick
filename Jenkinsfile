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
                        // Run Gitleaks to detect secrets
                        sh 'gitleaks detect --source . --report-format json --report-path gitleaks-report.json'
                        
                        // Prompt user for confirmation to proceed
                        def userInput = input(
                            id: 'proceedToNextStage',
                            message: 'Proceed to the next stage?',
                            parameters: [choice(choices: ['Yes', 'No'], description: 'Proceed to next stage?', name: 'PROCEED')]
                        )
                        
                        if (userInput == 'No') {
                            currentBuild.result = 'ABORTED'
                            error('Pipeline aborted by user choice.')
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Gitleaks found issues!'
                    }
                }
            }
        }

        stage('Verify mongodump Installation') {
            steps {
                script {
                    // Check if mongodump is installed
                    sh 'mongodump --version'
                }
            }
        }

        // Other stages as per your requirements
    }
}
