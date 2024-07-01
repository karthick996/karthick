pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        S3_BUCKET = "mongobackup0123"
        MONGO_SECRET_NAME = "mongo/creds"
        MONGO_HOST = "mdb.spanllc.com"
        MONGO_PORT = "27017"
        SLACK_CHANNEL = "#git-leaks-alerts" // Replace with your Slack channel        
        GITLEAKS_REPORT_FILE = 'gitleaks-report.json'  // Define the file to store Gitleaks report
    }

    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    def proceed = false
                    try {
                        // Run Gitleaks and capture the output
                        def status = sh(script: "gitleaks detect --source . --report-format json --report-path ${GITLEAKS_REPORT_FILE} > gitleaks-output.txt 2>&1", returnStatus: true)

                        // Read the Gitleaks output file
                        def outputFileContent = readFile('gitleaks-output.txt')

                        // Read the JSON report file
                        def reportContent = readFile(file: GITLEAKS_REPORT_FILE)
                        def parsedReport = readJSON text: reportContent

                        // Extract detailed findings from the report
                        def detailedFindings = parsedReport.collect { finding ->
                            """
                            **File:** ${finding.file}
                            **Line:** ${finding.line}
                            **Secret:** ${finding.secret}
                            **Rule:** ${finding.rule}
                            **Commit:** ${finding.commit}
                            **Date:** ${finding.date}
                            **Entropy:** ${finding.entropy}
                            **Author:** ${finding.author}
                            **Email:** ${finding.email}
                            **Message:** ${finding.message}
                            **Fingerprint:** ${finding.fingerprint}
                            """
                        }.join('\n\n')

                        // Format the output to be user-friendly
                        def formattedOutput = """
                        Gitleaks Scan Report:
                        ${outputFileContent}

                        Detailed Findings:
                        ${detailedFindings}
                        """

                        // Send Slack notification with Gitleaks output
                        slackSend(channel: env.SLACK_CHANNEL, color: '#FFFF00', message: formattedOutput)

                        // Prompt user for confirmation to proceed with Gitleaks findings
                        def userInput = input(
                            id: 'proceedToNextStage',
                            message: 'Gitleaks scan completed. Review the findings and decide whether to proceed.',
                            parameters: [
                                [$class: 'TextParameterDefinition', defaultValue: formattedOutput, description: 'Gitleaks findings', name: 'GITLEAKS_OUTPUT'],
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
                    // Check if mongodump is installed
                    sh 'mongodump --version'
                }
            }
        }

        // Add other stages as needed
    }
}
