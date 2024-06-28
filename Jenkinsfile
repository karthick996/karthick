pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        S3_BUCKET = "mongobackup0123"
        MONGO_SECRET_NAME = "mongo/creds"
        MONGO_HOST = "mdb.spanllc.com"
        MONGO_PORT = "27017"
        GITLEAKS_REPORT_FILE = 'gitleaks-report.json'  // Define the file to store Gitleaks report
    }
    
    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    try {
                        // Run Gitleaks to detect secrets and save report
                        sh "gitleaks detect --source . --report-format json --report-path ${GITLEAKS_REPORT_FILE}"
                        
                        // Read Gitleaks report to display in input step
                        def gitleaksOutput = readFile(file: "${GITLEAKS_REPORT_FILE}").trim()

                        // Prompt user for confirmation to proceed with Gitleaks findings
                        def userInput = input(
                            id: 'proceedToNextStage',
                            message: 'Gitleaks found issues. Proceed to the next stage?',
                            parameters: [
                                [$class: 'TextParameterDefinition', defaultValue: gitleaksOutput, description: 'Gitleaks findings', name: 'GITLEAKS_OUTPUT'],
                                [$class: 'ChoiceParameterDefinition', choices: ['Yes', 'No'], description: 'Proceed to next stage?', name: 'PROCEED']
                            ]
                        )
                        
                        // If user chooses not to proceed, abort the pipeline
                        if (userInput.PROCEED == 'No') {
                            currentBuild.result = 'ABORTED'
                            error('Pipeline aborted by user choice.')
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Error running Gitleaks or displaying input.'
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

        // Add other stages as needed
    }
}
