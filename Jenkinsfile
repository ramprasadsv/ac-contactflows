import groovy.json.JsonSlurper
import java.util.Map
import jenkins.*
import jenkins.model.*
import hudson.*
import hudson.model.* 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def ARN = "cd6d039c-604d-46b1-a567-01dedac27383"
def Instance_Alias = "jkewrnkds"
def Scheduled_Reports = '{"StorageType": "S3","S3Config": {"BucketName": "amazon-connect-92cff7c508d9","BucketPrefix": "connect/asdfdadsfd/Reports","EncryptionConfig":{"EncryptionType":"KMS","KeyId": "arn:aws:kms:us-east-1:357837012270:key/0725a969-88d2-4ad8-bd1c-69157746371e"}}}'
pipeline {
    agent any
    stages {
        stage('git repo & clean') {
            steps {
                   sh(script:"rm -r ac-contactflows", returnStdout: true)
                   sh(script:"git clone https://github.com/ramprasadsv/ac-contactflows.git", returnStdout: true)
                   sh(script:"ls -ltr", returnStdout: true)
            }
        }
        stage('install') {
            steps{
                echo 'Enabling S3 for storing scheduled reports'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                    //get Jenkins instance
                        def jenkins = Jenkins.instance
                    //get job Item
                        def item = jenkins.getItemByFullName("The_JOB_NAME")
                        println item
                    // get workspacePath for the job Item
                        def workspacePath = jenkins.getWorkspaceFor (item)
                        println workspacePath           
                        def jsonSlurper = new JsonSlurper()
                        //data = jsonSlurper.parse(new File(workspacePath.toString()+"\\instance.json"))
                        data = jsonSlurper.parse(new File("./instance.json"))
                        echo data
                        def sc = Scheduled_Reports
                        sc = sc.replaceAll('Instance_Alias', Instance_Alias)
                        echo sc
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type SCHEDULED_REPORTS --storage-config ${sc}", returnStdout: true).trim()
                        echo "Chat Transcripts : ${di}"
                    }
                }
            }
        }
        stage('test') {
            steps {
                echo "Testing "
            }
        }
        stage('package') {
            steps {
                echo "Package "
            }
        }
    }
}
