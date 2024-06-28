#!groovy
import jenkins.model.* 
import hudson.*
import hudson.model.*
import groovy.json.*
import java.text.SimpleDateFormat

def date = new Date()
today = new SimpleDateFormat("ddMMyyyy")
def COLOR_MAP = ['SUCCESS': '#00FF00', 'FAILURE': '#FF0000','UNSTABLE': '#FFFF00', 'ABORTED': '#800000']
def approverId
def slave
def envid
def highenv

pipeline {
    agent any

    environment {
        GITLEAKS_REPORT_FILE = 'gitleaks-report.json'  // Define the file to store Gitleaks report
    }

    stages {
        stage('Run Gitleaks') {
            steps {
                script {
                    def proceed = false
                    def gitleaksOutput = ''
                    try {
                        // Run Gitleaks and capture the output
                        def returnStatus = sh(script: "gitleaks detect --source . --report-format json --report-path ${GITLEAKS_REPORT_FILE} > gitleaks-output.txt 2>&1", returnStatus: true)

                        // Read the Gitleaks output file
                        def outputFileContent = readFile('gitleaks-output.txt')

                        // Format the output to be user-friendly
                        def formattedOutput = """
                            Gitleaks Scan Report:
                            ${outputFileContent}
                        """

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

        stage('MongoDump') {
            steps {
                script {
                    catlskeyfile = "/home/ubuntu/MongoDB/Keyfiles/"
                    mongodbbackup = "/opt/MongoDBBackup/"
                    sh "mongodump --ssl --uri 'mongodb+srv://mdb.spanllc.com' --sslPEMKeyFile '${catlskeyfile}/tls.pem' --sslCAFile '${catlskeyfile}/ca.cert' --port 27017 --username=TaxBandits_User --password=Azw8yNV15FGvgzN2 --authenticationDatabase=admin --gzip --archive='${mongodbbackup}/sample.zip'"
                }
            }
        }

        stage('Login and push to AWS S3') {
            steps {
                script {
                    s3 = "s3://span-devops/MongoDB/sample.zip"
                    sh "sudo aws s3 cp '${mongodbbackup}/sample.zip' '${s3}'"
                }
            }
        }
    }
}
