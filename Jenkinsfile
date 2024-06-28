pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        S3_BUCKET = "mongobackup0123"
        MONGO_SECRET_NAME = "mongo/creds"
        MONGO_HOST = "mdb.spanllc.com"
        MONGO_PORT = "27017"
        TARGET_REPO_URL = "https://gitlab.com/karthick1910421/karthick.git" // The URL of the repository to scan
    }

    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    try {
                        // Clone the target repository
                        sh "git clone ${TARGET_REPO_URL} target-repo"

                        // Change to the target repository directory
                        dir('target-repo') {
                            // Run Gitleaks
                            sh 'gitleaks detect --source . --report-format json --report-path gitleaks-report.json'
                        }

                        // Prompt for approval to proceed
                        def userInput = input message: 'Proceed to next stage?', ok: 'Proceed', parameters: [choice(choices: ['Proceed', 'Abort'], description: 'Proceed or Abort')]

                        // Check user input and handle accordingly
                        if (userInput == 'Abort') {
                            error 'Pipeline aborted by user'
                        }

                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Gitleaks found issues!'
                    }
                }
            }
        }

        stage('Next Stage') {
            when {
                expression {
                    // This stage will run only if 'Proceed' was chosen in the input step
                    return true
                }
            }
            steps {
                echo 'Continuing to next stage...'
                // Add your next stage steps here
            }
        }
    }
}
