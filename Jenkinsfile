pipeline {
    agent any

    environment {
        TARGET_REPO_URL = "https://github.com/karthick996/test-demo.git" // The URL of the repository to scan
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
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Gitleaks found issues!'
                    }
                }
            }
        }
    }
}

