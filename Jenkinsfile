pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        S3_BUCKET = "mongobackup0123"
        MONGO_SECRET_NAME = "mongo/creds"
        MONGO_HOST = "mdb.spanllc.com"
        MONGO_PORT = "27017"
        SLACK_CHANNEL = "#git-leaks-alerts"  // Replace with your Slack channel
        GITLEAKS_REPORT_FILE = 'gitleaks-report.json'  // Define the file to store Gitleaks report
    }

    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    def proceed = false
                    try {
                        // Run Gitleaks and capture the detailed output
                        def gitleaksStatus = sh(script: "gitleaks detect --source . --report-format json --report-path ${GITLEAKS_REPORT_FILE} > gitleaks-output.txt 2>&1", returnStatus: true)

                        // Check if Gitleaks command was successful
                        if (gitleaksStatus != 0) {
                            def gitleaksOutput = readFile('gitleaks-output.txt')
                            slackSend(channel: env.SLACK_CHANNEL, color: '#FF0000', message: "Gitleaks scan failed. Output:\n${gitleaksOutput}")
                            error("Gitleaks scan failed. See output above.")
                        }

                        // Read the detailed Gitleaks output file
                        def reportData = readJSON(file: 'gitleaks-report.json')
                        def detailedReport = reportData.findAll { it.severity == 'HIGH' || it.severity == 'MEDIUM' }
                        
                        // Prepare detailed report for Slack notification
                        def formattedOutput = detailedReport.collect { leak ->
                            """
                            *File:* ${leak.file}
                            *Line:* ${leak.line}
                            *Secret:* ${leak.secret}
                            *Type:* ${leak.type}
                            *Severity:* ${leak.severity}
                            """
                        }.join("\n\n")
                        
                        // Send detailed report to Slack
                        slackSend(channel: env.SLACK_CHANNEL, color: '#FF0000', message: "Gitleaks Detailed Scan Report:\n${formattedOutput}")

                        // Prepare user-friendly summary for user interaction
                        def summaryOutput = detailedReport.collect { leak ->
                            """
                            File: ${leak.file}
                            Type: ${leak.type}
                            Severity: ${leak.severity}
                            """
                        }.join("\n\n")

                        // Prompt user for confirmation to proceed with Gitleaks findings
                        def userInput = input(
                            id: 'proceedToNextStage',
                            message: 'Gitleaks scan completed. Review the summary and decide whether to proceed.',
                            parameters: [
                                [$class: 'TextParameterDefinition', defaultValue: summaryOutput, description: 'Gitleaks summary findings', name: 'GITLEAKS_SUMMARY'],
                                [$class: 'ChoiceParameterDefinition', choices: ['Yes', 'No'], description: 'Proceed to next stage?', name: 'PROCEED']
                            ]
                        )

                        // Set proceed variable based on user input
                        if (userInput['PROCEED'] == 'Yes') {
                            proceed = true
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo 'Error running Gitleaks or displaying input.'
                    }

                    // Check the proceed variable before moving to the next stage
                    if (!proceed) {
                        currentBuild.result = 'ABORTED'
                        error('Pipeline aborted by user choice.')
                    }
                }
            }
        }

        stage('Verify mongodump Installation') {
            steps {
                script {
                    sh 'mongodump --version'
                }
            }
        }

        // Add other stages as needed
    }
}

