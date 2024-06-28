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
                        sh 'gitleaks --path=. --report-format=json --report-path=gitleaks-report.json'
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
                    sh 'mongodump --version'
                }
            }
        }

        stage('Retrieve MongoDB Credentials') {
            steps {
                script {
                    // Retrieve MongoDB credentials from AWS Secrets Manager
                    def mongoCredentials = sh(script: """
                        aws secretsmanager get-secret-value --secret-id ${MONGO_SECRET_NAME} --region ${AWS_REGION} --query SecretString --output text
                    """, returnStdout: true).trim()

                    // Parse the JSON string manually
                    def mongoCredJson = new groovy.json.JsonSlurper().parseText(mongoCredentials)
                    env.MONGO_USER = mongoCredJson.username
                    env.MONGO_PASS = mongoCredJson.password

                    // Print retrieved credentials for debugging (Note: remove these lines in production)
                    echo "MongoDB User: ${env.MONGO_USER}"
                    echo "MongoDB Password: ${env.MONGO_PASS}"
                }
            }
        }

        stage('Backup MongoDB') {
            steps {
                script {
                    def date = new Date().format("yyyyMMddHHmmss")
                    def backupFileName = "mongo-backup-all-databases-${date}.gz"

                    // Perform backup of all databases
                    sh """
                        mongodump --uri="mongodb+srv://${env.MONGO_USER}:${env.MONGO_PASS}@${MONGO_HOST}/?replicaSet=mdb0&readPreference=secondaryPreferred" --ssl --sslAllowInvalidCertificates --archive=${backupFileName} --gzip
                    """

                    // Upload backup to S3
                    sh """
                        aws s3 cp ${backupFileName} s3://${S3_BUCKET}/${backupFileName} --region ${AWS_REGION}
                    """
                    
                    // Delete backup file from Jenkins server workspace
                    sh """
                        rm -f ${backupFileName}
                    """
                }
            }
        }

        stage('Cleanup Old Backups') {
            steps {
                script {
                    // List and sort backups by date, then delete all but the latest 3 in S3 bucket
                    def s3Files = sh(script: """
                        aws s3 ls s3://${S3_BUCKET}/ --region ${AWS_REGION} | awk '{print \$4}' | sort
                    """, returnStdout: true).trim().split('\n').findAll { it.startsWith('mongo-backup-all-databases-') }

                    echo "All backup files in S3: ${s3Files}"

                    if (s3Files.size() > 2) {
                        def filesToDelete = s3Files[0..(s3Files.size() - 3)]
                        echo "Files to delete: ${filesToDelete}"
                        filesToDelete.each { file ->
                            sh """
                                echo "Deleting ${file} from S3"
                                aws s3 rm s3://${S3_BUCKET}/${file} --region ${AWS_REGION}
                            """
                        }
                    } else {
                        echo "No old backups to delete. Total backups: ${s3Files.size()}"
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            echo 'Cleaning up...'
            deleteDir() // Clean workspace after build
        }
    }
}
