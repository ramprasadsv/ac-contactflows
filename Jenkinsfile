import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}
def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}
def CONTACTFLOW = ""

def INSTANCEARN = "662de594-7bab-4713-952b-2b4cb16f2724"
def FLOWID = "3b0db24a-c113-4847-8857-113c2c064131"

String TRAGETINSTANCEARN = "de1c040b-d1fe-4b12-b1e8-5e072329b86a"
String TARGETFLOWID = "733b11b2-42ec-42c2-9d20-ae657bc6a1e7"
String TARGETFLOWID2 = "082ffc0c-390f-4cd0-8480-231489f35618"
String TARGETJSON = ""
def PRIMARYLIST
def TARGETLIST
pipeline {
    agent any
    stages {
        stage('git repo & clean') {
            steps {
                   sh(script: "rm -r ac-contactflows", returnStdout: true)
                   sh(script: "git clone https://github.com/ramprasadsv/ac-contactflows.git", returnStdout: true)
                   sh(script: "ls -ltr", returnStatus: true)
                   
            }
        }
        
        stage('Populate Flows') {
            steps {
                echo "List all the flows in primary instance "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def ti =  sh(script: "aws connect list-contact-flows --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo ti
                        PRIMARYLIST = jsonParse(ti)
                        def si =  sh(script: "aws connect list-contact-flows --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo si
                        TARGETLIST = jsonParse(si)
                    }
                }
            }
        }
        
        stage('Find Missing flows') {
            steps {
                echo "Identify the flows missing and create them"
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        PRIMARYLIST.ContactFlowSummaryList.each {
                            println "item: $it.Name"
                            println "item: $it.ContactFlowType"
                        }
                    }
                }
            }
        }
        
    }
}
