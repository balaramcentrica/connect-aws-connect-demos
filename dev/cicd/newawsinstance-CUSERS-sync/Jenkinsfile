import groovy.json.JsonSlurper;
import groovy.json.JsonOutput; 

def CONFIGDETAILS 
String MISSINGLIST = ""
def INSTANCEARN = ""
def TRAGETINSTANCEARN = ""
String PRIMARYUSERS = ""
String TARGETUSERS = ""
String PRIMARYRPS = ""
String TARGETRPS = ""
String PRIMARYSECPROS = ""
String TARGETSECPROS = ""
String PRIMARYHRCHY = ""
String TARGETHRCHY = ""
String userName = ""
String pwd = ""
String firstName = ""
String lastName = ""
String email = ""
String idInfo = ""
String phoneType =""
String autoAccept = ""
String acw = ""
String dpn = ""
String pc =  ""
String rpId = ""
String spIds = ""
String hid = ""


pipeline {
    agent any
    parameters {
        string(name: 'TRAGET_INSTANCE',defaultValue:'',description:'AWS TARGET ID')
    }
    stages {
        stage('git repo & clean') {
            steps {
                script{
                   try{
                      sh(script: "rm -r user-syncronization-main", returnStdout: true)    
                   }catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                   }                   
                   sh(script: "git clone https://github.com/rambusnis/user-syncronization-main.git", returnStdout: true)
                   sh(script: "ls -ltr", returnStatus: true)
                   CONFIGDETAILS = sh(script: 'cat parameters.json', returnStdout: true).trim()
                   def config = jsonParse(CONFIGDETAILS)
                   INSTANCEARN = config.primaryInstance
                   //TRAGETINSTANCEARN = config.targetInstance
                   TRAGETINSTANCEARN = params.TRAGET_INSTANCE
                }
            }
        }
        
        stage('List all Resources SOURCE ') {
            steps {
                echo "List all Resources in both instance "
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'eu-west-2') {
                    script {
                        
                        PRIMARYUSERS =  sh(script: "aws connect list-users --instance-id ${INSTANCEARN} ", returnStdout: true).trim()
                        echo PRIMARYUSERS
                        
                        PRIMARYRPS =  sh(script: "aws connect list-routing-profiles --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYRPS
                        
                        PRIMARYSECPROS =  sh(script: "aws connect list-security-profiles --instance-id ${INSTANCEARN} ", returnStdout: true).trim()
                        echo PRIMARYSECPROS
                      
                        PRIMARYHRCHY = sh(script: "aws connect list-user-hierarchy-groups --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYHRCHY
                    }
                }
            }
        }
        
        stage('List all Resources TARGET ') {
            steps {
                echo "List all Resources in both instance "
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {
                    script {
                        
                        TARGETUSERS =  sh(script: "aws connect list-users --instance-id ${TRAGETINSTANCEARN} ", returnStdout: true).trim()
                        echo TARGETUSERS
                        
                        TARGETRPS =  sh(script: "aws connect list-routing-profiles --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETRPS 
                        
                        TARGETSECPROS =  sh(script: "aws connect list-security-profiles --instance-id ${TRAGETINSTANCEARN} ", returnStdout: true).trim()
                        echo TARGETSECPROS
                      
                        TARGETHRCHY = sh(script: "aws connect list-user-hierarchy-groups --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETHRCHY                      
                    }
                }
            }
        }
        stage('Find missing users') {
            steps {
                script {
                    echo "Find missing users in the target instance"
                    def pl = jsonParse(PRIMARYUSERS)
                    def tl = jsonParse(TARGETUSERS)
                    int listSize = pl.UserSummaryList.size() 
                    println "Primary list size $listSize"
                    for(int i = 0; i < listSize; i++){
                        def obj = pl.UserSummaryList[i]
                        String qcName = obj.Username
                        String qcId = obj.Id
                        boolean qcFound = checkList(qcName, tl)
                        if(qcFound == false) {
                            println "Missing Name : $qcName Id : $qcId"                                                              
                            MISSINGLIST = MISSINGLIST.concat(qcId).concat(",")                                
                        }
                    }
                }
                echo "Missing list in the target instance -> ${MISSINGLIST}"
            }
        }
        
        stage('Create the missing users') {
            steps {
                echo "Create the missing users in the target instance "                
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560',region: 'eu-west-2') {   
                    script {
                        if(MISSINGLIST.length() > 1 ){
                            def qcList = MISSINGLIST.split(",")
                            for(int i = 0; i < qcList.size(); i++){
                                String qcId = qcList[i]
                                if(qcId.length() > 2){
                                    def di =  sh(script: "aws connect describe-user --instance-id ${INSTANCEARN} --user-id ${qcId}", returnStdout: true).trim()
                                    echo di
                                    def user = jsonParse(di)
                                    
                                    userName = " --username " + user.User.Username
                                    pwd = "--password Connect@12312"                                    
                                    firstName = user.User.IdentityInfo.FirstName
                                    lastName = user.User.IdentityInfo.LastName
                                    email = user.User.IdentityInfo.Email                                    
                                    idInfo = " --identity-info " + "FirstName=" + firstName + ",LastName=" + lastName +",Email=" + email                                    
                                    phoneType = user.User.PhoneConfig.PhoneType
                                    autoAccept = user.User.PhoneConfig.AutoAccept
                                    acw = user.User.PhoneConfig.AfterContactWorkTimeLimit
                                    dpn = user.User.PhoneConfig.DeskPhoneNumber                                    
                                    pc = " --phone-config " + "PhoneType=" + phoneType + ",AutoAccept=" + autoAccept + ",AfterContactWorkTimeLimit=" + acw + ",DeskPhoneNumber=" + dpn                                     
                                    rpId = "--routing-profile-id " + getRPId(PRIMARYRPS, user.User.RoutingProfileId, TARGETRPS)                                    
                                    spIds = "--security-profile-ids " + getSPIds(PRIMARYSECPROS, user.User.SecurityProfileIds, TARGETSECPROS)
                                    hid = ""
                                    if(user.User.HierarchyGroupId != null) {
                                         hid = "--hierarchy-group-id " + getHrRchyId(PRIMARYHRCHY, user.User.HierarchyGroupId, TARGETHRCHY)
                                    }
                                    
                                    user = null

                                    withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560',region: 'us-west-2') {   
                                    script {
                                            def cq =  sh(script: "aws connect create-user ${userName} ${pwd} ${idInfo} ${pc} ${spIds} ${rpId} ${hid} --instance-id ${TRAGETINSTANCEARN} " , returnStdout: true).trim()
                                            echo cq

                                        }
                                    }
                                    
                                    
                               }
                            }
                        }
                    }                
                }
            }
        } 

         stage('user sync complete') {
            steps {
                echo "completed sycnronizing both instances"                
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {   
                    /*script{

                        def cq =  sh(script: "aws connect create-user ${userName} ${pwd} ${idInfo} ${pc} ${spIds} ${rpId} ${hid} --instance-id ${TRAGETINSTANCEARN} " , returnStdout: true).trim()
                        echo cq
                    }*/
                    echo "completed sycnronizing both instances"                
                }
            } 
         }
        
        
        
     }
}


@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

def checkList(qcName, tl) {
    boolean qcFound = false
    for(int i = 0; i < tl.UserSummaryList.size(); i++){
        def obj2 = tl.UserSummaryList[i]
        String qcName2 = obj2.Username
        if(qcName2.equals(qcName)) {
            qcFound = true
            break
        }
    }
    return qcFound
}

def getRPId (primary, searchId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for Id : $searchId"
    for(int i = 0; i < pl.RoutingProfileSummaryList.size(); i++){
        def obj = pl.RoutingProfileSummaryList[i]    
        if (obj.Id.equals(searchId)) {
            fName = obj.Name
            println "Found Name : $fName"
            break
        }
    }
    println "Searching for name : $fName"        
    for(int i = 0; i < tl.RoutingProfileSummaryList.size(); i++){
        def obj = tl.RoutingProfileSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found matching id : $rId"
            break
        }
    }
    return rId
}

def getSPIds (primary, searchIds, target) {
    String rId = ""
    searchIds.each{ sId -> 
        def pl = jsonParse(primary)
        def tl = jsonParse(target)
        String fName = ""
        println "Searching for Id : $sId"
        for(int i = 0; i < pl.SecurityProfileSummaryList.size(); i++){
            def obj = pl.SecurityProfileSummaryList[i]    
            if (obj.Id.equals(sId)) {
                fName = obj.Name
                println "Found Name : $fName"
                break
            }
        }
        println "Searching for name : $fName"        
        for(int i = 0; i < tl.SecurityProfileSummaryList.size(); i++){
            def obj = tl.SecurityProfileSummaryList[i]    
            if (obj.Name.equals(fName)) {
                rId = rId + " " + obj.Id
                println "Found matching id : $rId"
                break
            }
        }
    };
                  
    return rId
}

def getHrRchyId (primary, searchId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for Id : $searchId"
    for(int i = 0; i < pl.UserHierarchyGroupSummaryList.size(); i++){
        def obj = pl.UserHierarchyGroupSummaryList[i]    
        if (obj.Id.equals(searchId)) {
            fName = obj.Name
            println "Found Name : $fName"
            break
        }
    }
    println "Searching for name : $fName"        
    for(int i = 0; i < tl.UserHierarchyGroupSummaryList.size(); i++){
        def obj = tl.UserHierarchyGroupSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found matching id : $rId"
            break
        }
    }
    return rId
}
