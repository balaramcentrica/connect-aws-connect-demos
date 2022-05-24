import groovy.json.JsonSlurper;
import groovy.json.JsonSlurperClassic ;
import groovy.json.JsonOutput;
import groovy.json.*
import java.util.zip.*;
import java.util.Random;
import groovy.json.JsonBuilder
import groovy.io.FileType
import net.sf.json.groovy.JsonSlurper;


def CONFIGDETAILS 
String MISSINGQC = ""
def INSTANCEARN = ""
def TRAGETINSTANCEARN = ""
String PRIMARYQC = ""
String TARGETQC = ""
String PRIMARYQUEUES = ""
String TARGETQUEUES = ""
String PRIMARYCFS = ""
String TARGETCFS = ""
String PRIMARYHOP = ""
String TARGETHOP = ""

String qcName=""
String hopId=""
String maxContacts=""
String quickConnectConfig=""
String outBoundConfig=""
String hopName=""
String hopDescription=""
String hopTimeZone=""
String hopConfig=""
String fConfig=""

pipeline {
    agent any
    
    parameters {
        string(name:'STAGES',defaultValue:'',description:'Build Source')
        string(name:'SREGIONS',defaultValue:'',description:'Build Source')
        string(name:'TREGIONS',defaultValue:'',description:'Build Source')
        string(name:'TRAGET_INSTANCE',defaultValue:'',description:'Build Source')
    }
    
    stages {
        stage('git repo & clean') {
            steps {
                script{
                   try{
                      sh(script: "rm -r hours-syncronization-main", returnStdout: true)    
                   }catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                   }                   
                   sh(script: "git clone https://github.com/rambusnis/hours-syncronization-main.git", returnStdout: true)
                   sh(script: "ls -ltr", returnStatus: true)
                   CONFIGDETAILS = sh(script: 'cat parameters.json', returnStdout: true).trim()
                   def config = jsonParse(CONFIGDETAILS)
                   INSTANCEARN = "49782017-80ae-4b24-ab1d-ab2ec88a86d9"
                   TRAGETINSTANCEARN = params.TRAGET_INSTANCE
                   //TRAGETINSTANCEARN = "9edd958a-a59f-4ec4-956b-5cf28a132ab1" 
                   //hopConfig = config.confighop
                   //config.primaryInstance
                   //TRAGETINSTANCEARN = config.targetInstance                   
                }
            }
        }
        
        stage('List all Resources ') {
            steps {
                echo "List all Resources in both instance "
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'eu-west-2') {
                    script {
                        PRIMARYHOP = sh(script: "aws connect list-hours-of-operations --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYHOP
                    }
                }
            }
        }

        stage('List all Resources TARGET') {
            steps {
                echo "List all Resources in both instance "
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {
                    script {
                        TARGETHOP = sh(script: "aws connect list-hours-of-operations --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETHOP       
                
                    }
                }
            }
        }
        
        stage('Find missing queues') {
            steps {
                script {
                    echo "Find missing queues in the target instance"
                    def pl = jsonParse(PRIMARYHOP)
                    def tl = jsonParse(TARGETHOP)
                    int listSize = pl.HoursOfOperationSummaryList.size() 
                    println "Primary list size $listSize"
                    for(int i = 0; i < listSize; i++){
                        def obj = pl.HoursOfOperationSummaryList[i]
                        qcName = obj.Name
                        String qcId = obj.Id

                        echo "${qcName}"
                        echo "${qcId}"
                        boolean qcFound = checkList(qcName, tl)
                        if(qcFound == false) {
                            println "Missing HRS Name : $qcName Id : $qcId"                                                              
                            MISSINGQC = MISSINGQC.concat(qcId).concat(",")                                
                        }
                    }
                }
                echo "Missing list in the target instance -> ${MISSINGQC}"
            }
        }
      
      
        stage('Create the missing HRSOPR') {
            steps {
                echo "Create the missing HRSPOS in the target instance "                
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'eu-west-2') {   
                    script {
                        def di=""
                        def parsehop=""
                        String json=""
                        
                        if(MISSINGQC.length() > 1 ){
                            def qcList = MISSINGQC.split(",")
                            for(int i = 0; i < qcList.size(); i++){
                                String qcId = qcList[i]
                                if(qcId.length() > 2){
                                    di =  sh(script: "aws connect  describe-hours-of-operation --instance-id ${INSTANCEARN} --hours-of-operation-id ${qcId} --output json ${' > '}${qcId}${'.json'}", returnStdout: true).trim()
                                    echo di
                                    def CONFIGDETAILS1 = sh(script: "${'cat '}${qcId}${'.json'}" , returnStdout: true).trim()
                                    def config1 = jsonParse(CONFIGDETAILS1)
                                    json = config1.HoursOfOperation.Config
                                    echo "${json}"
                                    json = json.replace ('=',':')
                                    json = json.replace ('Day','"Day"')
                                    json = json.replace ('Hours','"Hours"')
                                    json = json.replace ('Minutes','"Minutes"')
                                    json = json.replace ('EndTime','"EndTime"')
                                    json = json.replace ('StartTime','"StartTime"')
                                    json = json.replace ('MONDAY','"MONDAY"')
                                    json = json.replace ('TUESDAY','"TUESDAY"')
                                    json = json.replace ('WEDNESDAY','"WEDNESDAY"')
                                    json = json.replace ('THURSDAY','"THURSDAY"')
                                    json = json.replace ('FRIDAY','"FRIDAY"')
                                    json = json.replace ('SATURDAY','"SATURDAY"')
                                    json = json.replace ('SUNDAY','"SUNDAY"')
                                    json = json.replace('"','\"')

                                    def json1= toJSON(json)
                                    echo "${json1}"

                                    withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {
                                    script{
                                    def dic =  sh(script: "aws connect create-hours-of-operation --instance-id ${TRAGETINSTANCEARN} --name ${config1.HoursOfOperation.Name} --description ${config1.HoursOfOperation.Description} --time-zone ${config1.HoursOfOperation.TimeZone} --config ${json1}", returnStdout: true).trim()
                                    echo "${dic}"
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
}


@NonCPS

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

def toSTRINGTOJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

def toJSON2(def json) {
    JsonOutput.prettyPrint(JsonOutput.toJson(json))
}

def checkList(qcName, tl) {
    boolean qcFound = false
    for(int i = 0; i < tl.HoursOfOperationSummaryList.size(); i++){
        def obj2 = tl.HoursOfOperationSummaryList[i]
        String qcName2 = obj2.Name
        if(qcName2.equals(qcName)) {
            qcFound = true
            break
        }
    }
    return qcFound
}

def getFlowId (primary, flowId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for flowId : $flowId"
    for(int i = 0; i < pl.ContactFlowSummaryList.size(); i++){
        def obj = pl.ContactFlowSummaryList[i]    
        if (obj.Id.equals(flowId)) {
            fName = obj.Name
            println "Found flow name : $fName"
            break
        }
    }
    println "Searching for flow name : $fName"        
    for(int i = 0; i < tl.ContactFlowSummaryList.size(); i++){
        def obj = tl.ContactFlowSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found flow id : $rId"
            break
        }
    }
    return rId
}

def getQuickConnectId (primary, name, target) {
    echo "come inside getQuickConnectId"
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = name
    String rId = ""
    echo "Find for name : ${fName}"       
    echo "tl.QuickConnectSummaryList.size()  : ${tl.QuickConnectSummaryList.size()}"
    for(int i = 0; i < tl.QuickConnectSummaryList.size(); i++){
        echo "getQuickConnectId quick connect arns count : ${i}"        
        def obj = tl.QuickConnectSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found id : $rId"
            break
        }
    }
    return rId
}

def getHopId (primary, userId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for userId : $userId"
    for(int i = 0; i < pl.HoursOfOperationSummaryList.size(); i++){
        def obj = pl.HoursOfOperationSummaryList[i]    
        if (obj.Id.equals(userId)) {
            fName = obj.Name
            println "Found name : $fName"
            break
        }
    }
    println "Searching for Id for : $fName"        
    for(int i = 0; i < tl.HoursOfOperationSummaryList.size(); i++){
        def obj = tl.HoursOfOperationSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found Id : $rId"
            break
        }
    }
    return rId
 }
