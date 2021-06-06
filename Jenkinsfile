import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def CONTACTFLOW = ""
def INSTANCEARN = "662de594-7bab-4713-952b-2b4cb16f2724"
def FLOWID = "3b0db24a-c113-4847-8857-113c2c064131"
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
        stage('read contact flow') {
            steps{
                echo 'Reading the contact flow content '
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        //def di =  sh(script: "aws connect describe-contact-flow --instance-id ${INSTANCEARN} --contact-flow-id ${FLOWID}", returnStdout: true).trim()
                        //echo di
                        //def data2 = sh(script: 'cat contactflow.json', returnStdout: true).trim()    
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
                            println(content.indexOf(arnmapping[i].sourceARN, 1))
                            content = content.replaceAll(arnmapping[i].sourceARN, arnmapping[i].targetARN)
                        }
                        echo content                        
                        def json = JsonOutput.toJson(content)
                        echo json
                    }
                }
            }
        }
        stage('update contact flow') {
            steps {
                echo "Testing "
            }
        }
        stage('end') {
            steps {
                echo "Completed "
            }
        }
    }
}
