import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}
def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}
def checkList(qcName, tl) {
    boolean qcFound = false
    for(int i = 0; i < tl.QuickConnectSummaryList.size(); i++){
        def obj2 = tl.QuickConnectSummaryList[i]
        String qcName2 = obj2.Name
        if(qcName2.equals(qcName)) {
            qcFound = true
        }
    }
    return qcFound
}


def INSTANCEARN = "662de594-7bab-4713-952b-2b4cb16f2724"
def FLOWID = "3b0db24a-c113-4847-8857-113c2c064131"
//def MISSINGQC = [:]
String MISSINGQC = ""
String TRAGETINSTANCEARN = "de1c040b-d1fe-4b12-b1e8-5e072329b86a"
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
                        def pl = jsonParse(PRIMARYLIST)
                        def tl = jsonParse(TARGETLIST)
                        int listSize = pl.QuickConnectSummaryList.size() 
                        println "Primary list size $listSize"
                        for(int i = 0; i < listSize; i++){
                            def obj = pl.QuickConnectSummaryList[i]
                            String qcName = obj.Name
                            String qcId = obj.Id
                            String qcType = obj.QuickConnectType
                            boolean qcFound = checkList(qcName, tl)
                            if(qcFound == false) {
                                println "Missing $qcName of type : $qcType -> $qcId"                                                              
                                MISSINGQC = MISSINGQC.concat(qcId).concat(",")                                
                            }
                        }
                        echo "Missing list -> ${MISSINGQC}"
                    }
                }
            }
        }
        
        stage('Find Missing quick connects') {
            steps {
                echo "Identify the quick connects "                
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {   
                    script {
                        def qcList = MISSINGQC.split(",")
                        for(int i = 0; i < qcList.size(); i++){
                            String qcId = qcList[i]
                            if(qcId.length() > 2){
                                def di =  sh(script: "aws connect describe-quick-connect --instance-id ${INSTANCEARN} --quick-connect-id ${qcId}", returnStdout: true).trim()
                                echo di
                                def qc = jsonParse(di)
                                String qcConfig=""
                                if(qc.QuickConnect.QuickConnectConfig.QuickConnectType.equals("PHONE_NUMBER")){
                                    qcConfig = '"QuickConnectConfig":{"QuickConnectType":"PHONE_NUMBER","PhoneConfig":{"PhoneNumber":"${qc.QuickConnect.QuickConnectConfig.PhoneConfig.PhoneNumber"}}}'
                                }else if(qc.QuickConnect.QuickConnectConfig.QuickConnectType.equals("USER")){
                                    //qcConfig = '"QuickConnectConfig":{"QuickConnectType":"USER","UserConfig":{"UserId":"${qc.QuickConnect.QuickConnectConfig.PhoneConfig.PhoneNumber"}}}'
                                }else{
                                    //qcConfig = '"QuickConnectConfig":{"QuickConnectType":"QUEUE","QueueConfig":{"QueueId":"${qc.QuickConnect.QuickConnectConfig.PhoneConfig.PhoneNumber"}}}'
                                    qc = null
                                    def dq =  sh(script: "aws connect describe-queue --instance-id ${INSTANCEARN} --queue-id ${qc.QuickConnect.QuickConnectConfig.QueueConfig.QueueId}", returnStdout: true).trim()
                                    echo dq
                                    def dc =  sh(script: "aws connect describe-contact-flow --instance-id ${INSTANCEARN} --contact-flow-id ${qc.QuickConnect.QuickConnectConfig.QueueConfig.ContactFlowId}", returnStdout: true).trim()
                                    echo dc
                                }
                                echo qcConfig
                                //def cq =  sh(script: "aws connect create-quick-connect --instance-id ${INSTANCEARN} --name ${qc.QuickConnect.Name} --description ${qc.QuickConnect.Description} --quick-connect-config ", returnStdout: true).trim()
                            }
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
