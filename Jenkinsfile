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
    agent {
        label 'Build-Nodejs'
    }

 

stages {
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
