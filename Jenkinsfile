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
String PRIMARYLIST = ""
String TARGETLIST = ""

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
        
        stage('Primary instance') {
            steps {
                echo "List all the flows in both instance "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        PRIMARYLIST =  sh(script: "aws connect list-contact-flows --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYLIST
                        TARGETLIST =  sh(script: "aws connect list-contact-flows --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETLIST 
                    }
                }
            }
        }
        
        stage('Find Missing flows') {
            steps {
                echo "Identify the flows missing and create them"
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def pl = jsonParse(PRIMARYLIST)
                        pl.ContactFlowSummaryList.each {
                            println "Checking flow : $it.Name of Type $it.ContactFlowType"
                            def flowName = $it.Name
                            def flowType = $it.ContactFlowType
                            def flowId = $it.Id
                            def tl = jsonParse(TARGETLIST)
                            def flowFound = false
                            tl.ContactFlowSummaryList.each {
                                def fn = $it.Name
                                if(flowName.equals(fn)) {
                                    flowFound = true
                                    println "Found the flow $flowName"
                                }
                            }
                            if(flowFound == false) {
                               withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                                   script {
                                        def di =  sh(script: "aws connect describe-contact-flow --instance-id ${INSTANCEARN} --contact-flow-id ${flowId}", returnStdout: true).trim()
                                        echo di
                                    }
                                }
                            }
                        }
                        
                    }
                }
            }
        }
        
    }
}
