import groovy.json.JsonSlurper;
import groovy.json.JsonSlurperClassic ;
import groovy.json.JsonOutput;

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

def TRAGET_INSTANCE_ID=""

pipeline { 
    agent any 
    
    parameters {
        choice(choices: ['DEV', 'STAGE','PROD'], description: 'AWS ENVIRONMENT?', name: 'PickAnStage')
        choice(choices: ['eu-west-2'], description: 'AWS SORUCE REGION?', name: 'PickAnSRCRegion')
        choice(choices: ['us-west-2'], description: 'AWS TARGET REGION?', name: 'PickAnTARRegion')
    }
    
        stages {
            stage('Initial-Connect-ExpImp-CreateNewInstance') { 
                steps { 
                    script{
                        build job: 'AWS-Connect-ExpImp-CreateNewInstance',parameters:[
                                                                            string(name:'STAGES',value:params.PickAnStage),
                                                                            string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                            string(name:'TREGIONS',value:params.PickAnTARRegion)
                                                                            ]                
                    }
                }
        }
        
        stage('Connect-HOUROPS-Sync'){
            steps {
                script{

                    TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                    echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                    build job: 'AWS-Connect-HRSOPS-Sync',parameters: [
                                                                     string(name:'TRAGET_INSTANCE',value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion)
                                                                     ]  
                }
            }
        }
        
        stage('Connect-Queue-Sync'){
            steps {
                script{
                    TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                    echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                    build job: 'AWS-Connect-Queue-Sync',parameters: [
                                                                     string(name:'TRAGET_INSTANCE', value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion),
                                                                     ]   
                }
            }
        }
        
        stage('RoutingProfile-Sync') {
            steps {
                 script{
                    TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                    echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                    build job: 'AWS-Connect-RoutingProfile-Sync',parameters: [
                                                                     string(name: 'TRAGET_INSTANCE', value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion)
                                                                     ]  
                }
            }
        }
        
        stage('Users-Sync') {
            steps {
                script{
                TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                build job: 'AWS-Connect-Users-Sync',parameters: [
                                                                     string(name: 'TRAGET_INSTANCE', value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion),
                                                                ]   
                }
            }
        }
        
        stage('QuickConnect-Sync') {
            steps {
                 script{
                    TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                    echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                    build job: 'AWS-Connect-QuickConnect-Sync',parameters: [
                                                                     string(name: 'TRAGET_INSTANCE', value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion)
                                                                     ]  
                }
            }
        }

        stage('ContactFlow-Sync') {
            steps {
                 script{
                    TRAGET_INSTANCE_ID= sh(script: 'cat /var/lib/jenkins/newconnectInstance.txt', returnStdout: true).trim()
                    echo "TRAGET_INSTANCE_ID    " + TRAGET_INSTANCE_ID
                    build job: 'AWS-Connect-ContactFlow-Sync',parameters: [
                                                                     string(name: 'TRAGET_INSTANCE', value:TRAGET_INSTANCE_ID),
                                                                     string(name:'STAGES',value:params.PickAnStage),
                                                                     string(name:'SREGIONS',value:params.PickAnSRCRegion),
                                                                     string(name:'TREGIONS',value:params.PickAnTARRegion)
                                                                     ]  
                        }
                }
        }
        
    }
}
