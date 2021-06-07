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
        stage('read flow from git') {
            steps{
                echo 'Reading the contact flow content '
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def data = sh(script: 'cat a-test1.json', returnStdout: true).trim()    
                        echo data
                        def data2 = sh(script: 'cat arnmapping.json', returnStdout: true).trim()    
                        echo data2
                        def flow = jsonParse(data)
                        def arnmapping = jsonParse(data2)
                        String content = flow.ContactFlow.Content    
                        echo content
                        for(i = 0; i < arnmapping.size(); i++){
                            echo "Checking on ARN : ${arnmapping[i].sourceARN}"
                            content = content.replaceAll(arnmapping[i].sourceARN, arnmapping[i].targetARN)
                        }
                        echo content                        
                        String json = toJSON(content)
                        echo json.toString()
                        println( json.getClass() )
                        TARGETJSON = json.toString()                        
                    }
                }
            }
        }
        stage('deploy flow after reading from git') {
            steps {
                echo "updating flow content after reading from git "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def di =  sh(script: "aws connect update-contact-flow-content --instance-id ${TRAGETINSTANCEARN} --contact-flow-id ${TARGETFLOWID} --content ${TARGETJSON}", returnStdout: true).trim()
                        echo di
                    }
                }
            }
        }
        stage('read flow from api') {
            steps {
                    echo 'Reading the contact flow content via api'
                    withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                        script {
                            def di =  sh(script: "aws connect describe-contact-flow --instance-id ${INSTANCEARN} --contact-flow-id ${FLOWID}", returnStdout: true).trim()
                            echo di
                            def data2 = sh(script: 'cat arnmapping.json', returnStdout: true).trim()    
                            def flow = jsonParse(di)
                            def arnmapping = jsonParse(data2)
                            String content = flow.ContactFlow.Content    
                            for(i = 0; i < arnmapping.size(); i++){
                                content = content.replaceAll(arnmapping[i].sourceARN, arnmapping[i].targetARN)
                            }
                            String json = toJSON(content)
                            TARGETJSON = json.toString()
                     }
                }
            }
        }
        
        stage('deploy updated flow after api') {
            steps {
                echo "Updating contact flow after reading from api "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def di =  sh(script: "aws connect update-contact-flow-content --instance-id ${TRAGETINSTANCEARN} --contact-flow-id ${TARGETFLOWID2} --content ${TARGETJSON}", returnStdout: true).trim()
                        echo di
                    }
                }
            }
        }
        
        
        stage('resolve missing contact flows') {
            steps {
                echo "List all the flows in both instances "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def ti =  sh(script: "aws connect list-contact-flows --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo ti
                        def si =  sh(script: "aws connect list-contact-flows --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo si
                        
                    }
                }
            }
        }
        
    }
}
