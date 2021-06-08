import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}
def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}
def checkList(primaryList, targetList) {
    def pl = jsonParse(primaryList)
    def tl = jsonParse(targetList)
    def map = [:]
    //Map<String,String> map =new HashMap()
    for(int j = 0; j < tl.QuickConnectSummaryList.size(); j++){
        def obj = pl.QuickConnectSummaryList[j]
        String qcName = obj.Name
        String qcId = obj.Id
        String qcType = obj.QuickConnectType
        boolean qcFound = false
        for(int i = 0; i < tl.QuickConnectSummaryList.size(); i++){
            def obj2 = tl.QuickConnectSummaryList[i]
            String qcName2 = obj.Name
            if(qcName2.equals(qcName)){
                flowFound = true
            }
        }
        if(qcFound == false){
           println "Not able to find : $qcId with Name -> $qcName"
           String qc = qcType.concat("#").concat(qcName)
           map.put(qcId.toString(), qc)
        }
    }
    return map
}

def awsAction (arn, flowId) {
    def di
    withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
       di =  sh(script: "aws connect describe-contact-flow --instance-id ${arn} --contact-flow-id ${flowId}", returnStdout: true).trim()
       echo di                                     
    }
    return di
}

def getFlowContent(flow) {
    def fd = jsonParse(flow)
    String content = fd.ContactFlow.Content
    String json = toJSON(content)
    content = json.toString()
    return content
}

def CONTACTFLOW = ""

def INSTANCEARN = "662de594-7bab-4713-952b-2b4cb16f2724"
def FLOWID = "3b0db24a-c113-4847-8857-113c2c064131"
def MISSINGQC = [:]
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
        
        stage('List all quick connects') {
            steps {
                echo "List all quick in both instance "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        PRIMARYLIST =  sh(script: "aws connect list-quick-connects --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYLIST
                        TARGETLIST =  sh(script: "aws connect list-quick-connects --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETLIST 
                        MISSINGQC = checkList(PRIMARYLIST, TARGETLIST)
                        echo MISSINGQC
                    }
                }
            }
        }
        
        stage('Find Missing quick connects') {
            steps {
                echo "Identify the flows quick connects "                
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {   
                    script {
                        MISSINGQC.each { key ->
                            def qcId = key
                            def di =  sh(script: "aws connect describe-quick-connect --instance-id ${INSTANCEARN} --quick-connect-id ${qcId}", returnStdout: true).trim()
                            echo di
                        }
                    }                
                }
            }
        } 

         stage('Create Missing quick connects') {
            steps {
                echo "Create the quick connects that were missing"                
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {   
                }
            } 
         }
        
     }
}
