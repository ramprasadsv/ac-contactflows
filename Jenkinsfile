import groovy.json.JsonSlurper


@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def ARN = "cd6d039c-604d-46b1-a567-01dedac27383"

pipeline {
    agent any
    stages {
        stage('git repo & clean') {
            steps {
                   //sh(script:"rmdir  /s /q ac-contactflows", returnStdout: true)
                   sh(script:"git clone https://github.com/ramprasadsv/ac-contactflows.git", returnStdout: true)
            }
        }
        stage('install') {
            steps{
                echo 'Enabling S3 for storing scheduled reports'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def sc = params.Scheduled_Reports
                        sc = sc.replaceAll('Instance_Alias', params.Instance_Alias)
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
