import groovy.json.JsonSlurper;
import groovy.json.JsonSlurperClassic ;
import groovy.json.JsonOutput;
import groovy.json.*
import java.util.zip.*;
import java.util.Random;
import groovy.json.JsonBuilder
import groovy.io.FileType
import net.sf.json.groovy.JsonSlurper;

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

def giveMeKey(){
    def key
    String alphabet = (('A'..'N')+('P'..'Z')+('a'..'k')+('m'..'z')+('2'..'9')).join() 
    def length = 12
    key = new Random().with {
        (1..length).collect { alphabet[ nextInt( alphabet.length() ) ] }.join()
        }
    return key
}


String AWS_CONFIG_LAMCONFIGFILE=""
String AWS_CONFIG_LEXCONFIGFILE=""
String AWS_CONFIG_KINCONFIGFILE=""
String AWS_CONFIG_KFNCONFIGFILE=""
String AWS_CONFIG_DYBCONFIGFILE=""
String INSTANCEALIAS = "prd-enkin-amzn-cog-dev"
String ENABLEINBOUNDCALLS = "true"
String ENABLEOUTBOUNDCALLS = "true"
String CONTACTFLOWLOGS = "true"
String IDENTITYMANAGEMENTTYPE = "CONNECT_MANAGED"
String APPROVEDORIGINS = "https://www.amazon.com,https://www.aws.com"
String CONTACTTRACERECORDS = ""
String AGENTEVENTS = ""
String CALLRECORDINGS = ""
String CHATTRANSCRIPTS = ""
String CHATATTACHMENTS = ""
String SCHEDULEDREPORTS = ""
String CONFIGDETAILS = ""
String ARN = ""
String TRAGET_INSTANCE_ID = ""
def AWS_CONFIG_S3BAWSCALLRECORD=""
def AWS_CONFIG_S3BAWSCHATTRANSCRIPT = ""
def AWS_CONFIG_S3BAWSREPORTS = ""
def AWS_CONFIG_S3BAWSAGENTEVENTS = ""


pipeline {
    
    agent any
    
    parameters {
        string(name:'STAGES',defaultValue:'',description:'Build Source')
        string(name:'SREGIONS',defaultValue:'',description:'Build Source')
        string(name:'TREGIONS',defaultValue:'',description:'Build Source')
        string(name:'TRAGET_INSTANCE',defaultValue:'',description:'Build Source')
    }

    stages {

    // Lamda Function Names
    // S3 Bucket Names
    // Lex BOT Names
    // Kinesis Stream Names
    // Kinesis Firehose Names
    // DynamoDB Table Names
    // API GW REST Names
    // Amplify WEB Site    

    
    stage('Init Jenkins Jobs') {
        steps {
            script{
                
                try{
                      sh(script: "rm -r connect-aws-connect-demos", returnStdout: true)    
                   }catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                   }   
                sh(script: "git clone https://github.com/Connect-Managed-Services/connect-aws-connect-demos.git", returnStdout: true)
                sh(script: "ls -ltr", returnStatus: true)
            }
        }
    }      

    stage('Export Lambdas/S3/LEX/DynamoDB/APIGW/AMPLIFY - Source Origin'){
        steps {
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.SREGIONS) {
                script {

                    // LAMDAS 
                    //****************************************************************************   
                    sh'''
                        #!/bin/bash
                        dldir="./lam_functions"
                        [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                        [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    '''
                    
                    sleep(time:3,unit:"SECONDS")

                    PRIMARYLAMBDAS = sh(script: "aws connect list-lambda-functions --instance-id ${"49782017-80ae-4b24-ab1d-ab2ec88a86d9"}", returnStdout: true).trim()
                    echo PRIMARYLAMBDAS
                    def p1 =  new groovy.json.JsonSlurperClassic().parseText(PRIMARYLAMBDAS)                        
                    def pl = p1.LambdaFunctions
                    
                    String env =""
                    String des=""
                    def env1=""
                    def des1=""
                    for(int i = 0; i < pl.size(); i++){
                        def obj = pl[i].split(":")
                        try{
                            PRIMARYLAMBDAS1 = sh(script: "aws lambda get-function-configuration --function-name ${obj[6]}", returnStdout: true).trim()
                            echo PRIMARYLAMBDAS1
                                def p2 =  new groovy.json.JsonSlurperClassic().parseText(PRIMARYLAMBDAS1)  
                                if (p2.Environment == null || p2.Environment == "")
                                { 
                                    env = "{\"Variables\":{\"env\":\"PROD\"}}"
                                    env1 = env
                                }
                                else{
                                    env = toJSON(p2.Environment)
                                    env=env.replace('"','\"')
                                    echo " STRREPLACE :  ${env}"
                                    env1 = env
                                }
                                
                                def expshret =sh(script: "aws lambda get-function --function-name ${obj[6]} --query 'Code.Location' | xargs wget -O ${'./lam_functions'}/${obj[6]}${'.zip'}", returnStdout: true).trim()
                                echo "${expshret}"

                            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                                script{
                                    String impshret =  sh(script: "aws lambda create-function --function-name ${obj[6]} --runtime ${p2.Runtime} --zip-file ${'fileb:///'}${WORKSPACE}/${'lam_functions'}/${obj[6]}${'.zip'} --handler ${p2.Handler} --role ${p2.Role} --timeout ${p2.Timeout} --memory-size ${p2.MemorySize} --environment ${toJSON(env1)}", returnStdout: true).trim()
                                    def impshret1 = new groovy.json.JsonSlurperClassic().parseText(impshret)
                                    AWS_CONFIG_LAMCONFIGFILE = AWS_CONFIG_LAMCONFIGFILE + impshret1.FunctionArn + ','
                                }
                            }

                        }catch(Exception ex) {
                            println("Catching the exception");
                        }
                    
                    } 




                    //LAMBDAENVEXPORTFILE= sh(script: 'cat LamFNameList.json', returnStdout: true).trim()
                    //def expparsedJson = new groovy.json.JsonSlurperClassic().parseText(LAMBDAENVEXPORTFILE)
                    //for(int expi = 0; expi < expparsedJson.size(); expi++){
                    //    def expshret =sh(script: "aws lambda get-function --function-name ${expparsedJson[expi].lamname} --query 'Code.Location' | xargs wget -O ${'./lam_functions'}/${expparsedJson[expi].lamname}${'.zip'}", returnStdout: true).trim()
                    //    echo "${expshret}" 
                    //} 


                    //****************************************************************************   

                    //LEX BOTS
                    //****************************************************************************                       
                    sh'''
                        #!/bin/bash
                        dldir="./lex_functions"
                        [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                        [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    '''
                    sleep(time:3,unit:"SECONDS")
                    
                    sh'''
                        #!/bin/bash
                        items=`cat LexFNameList.json`  
                        arr=$(echo "$items" | jq -c -r '.[]')
                        # iterate through the Bash array
                        for item in ${arr[@]}; do
                            lexname=$(echo $item | jq -r '.lexname')
                            lextype=$(echo $item | jq -r '.lextype')
                            lexexptype=$(echo $item | jq -r '.lexexptype')
                            lexversion=$(echo $item | jq -r '.lexversion')
                            lexSregion=$(echo $item | jq -r '.lexSregion')
                            lexTregion=$(echo $item | jq -r '.lexTregion')
                            lexoverride=$(echo $item | jq -r '.lexoverride')

                            url=$(aws lex-models get-export --name $lexname --resource-type $lextype --export-type $lexexptype --resource-version $lexversion --region $lexSregion --output json | grep url | cut -d '"' -f4)
                            wget $url -O ./lex_functions/$lexname.zip                           
                        done
                    '''     
                    //****************************************************************************
                    
                    //DYNAMODB's
                    //**************************************************************************** 

                    sh'''
                        #!/bin/bash
                        dldir="./dyb_functions"
                        [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                        [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    '''
                    sleep(time:3,unit:"SECONDS")

                    DYNAMODBIMPORTFILE= sh(script: 'cat DybFNameList.json', returnStdout: true).trim()
                    def dybparsedJson = new groovy.json.JsonSlurperClassic().parseText(DYNAMODBIMPORTFILE)
                    for(int dybi = 0; dybi < dybparsedJson.size(); dybi++){

                        EXPORTDYNAMDBOFILE = sh(script:"${"aws dynamodb scan --table-name ${dybparsedJson[dybi].dbtname} --region ${dybparsedJson[dybi].dbtsregion} --select ${dybparsedJson[dybi].dbtattribute} --max-items ${dybparsedJson[dybi].dbtmaxitems} --output json ${' > '}${'./'}${'dyb_functions'}/${dybparsedJson[dybi].dbtname}${'.json'}"}", returnStdout: true).trim()
                        echo "${EXPORTDYNAMDBOFILE}"

                        def CREATEDYNAMDBOFILE = sh(script: "aws dynamodb create-table --table-name ${dybparsedJson[dybi].dbtname} --region ${dybparsedJson[dybi].dbttregion} --attribute-definitions AttributeName=${dybparsedJson[dybi].dbtattributedef},AttributeType=S --key-schema AttributeName=${dybparsedJson[dybi].dbtattributekey},KeyType=HASH  --billing-mode ${dybparsedJson[dybi].dbtbillmode}", returnStdout: true).trim()
                        echo "${CREATEDYNAMDBOFILE}"                            

                        def DYNAMODBREADTABLEEXIST= sh(script: "aws dynamodb wait table-exists --table-name ${dybparsedJson[dybi].dbtname} --region ${dybparsedJson[dybi].dbttregion}", returnStdout: true).trim()
                        echo "${DYNAMODBREADTABLEEXIST}"                            

                        DYNAMODBREADJSONFILE= sh(script: "cat ${WORKSPACE}/${"dyb_functions/"}${dybparsedJson[dybi].dbtname}${'.json'}", returnStdout: true).trim()
                        def dybparsedJsonFile = new groovy.json.JsonSlurperClassic().parseText(DYNAMODBREADJSONFILE)
                        for ( int dybj=0 ;dybj < dybparsedJsonFile.size();dybj++)
                        {
                            def PUTITEMDYNAMDBOFILE = sh(script: "aws dynamodb put-item --region ${dybparsedJson[dybi].dbttregion} --table-name ${dybparsedJson[dybi].dbtname} --item ${"'"}${toJSON(dybparsedJsonFile['Items'][dybj])}${"'"}", returnStdout: true).trim()
                            echo "${PUTITEMDYNAMDBOFILE}"  
                        }
                    }  
                    //**************************************************************************** 

                    //API GW
                    //**************************************************************************** 

                    sh'''
                        #!/bin/bash
                        dldir="./apigw_funcitons"
                        [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                        [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    '''
                    sleep(time:3,unit:"SECONDS")

                    APIGWEXPORTFILE= sh(script: 'cat ApgFNameList.json', returnStdout: true).trim()
                    def apgparsedexpJson = new groovy.json.JsonSlurperClassic().parseText(APIGWEXPORTFILE)
                    for(int apgi = 0; apgi < apgparsedexpJson.size(); apgi++){

                        def APIGWCONFIGEXPNAME = sh(script: "aws apigateway get-rest-api --rest-api-id ${apgparsedexpJson[apgi].apiId}", returnStdout: true).trim()
                        def apigwName = jsonParse(APIGWCONFIGEXPNAME)

                        def APIGWEXPCONFIG = sh(script: "aws apigateway get-export --parameters ${'extensions='}${'''integrations'''} --rest-api-id ${apgparsedexpJson[apgi].apiId} --stage ${apgparsedexpJson[apgi].apiSstage} --export-type ${apgparsedexpJson[apgi].apiexptype} ${WORKSPACE}/${"apigw_funcitons/"}${apgparsedexpJson[apgi].apiname}${'.json'}", returnStdout: true).trim()
                        def lapigwconfig = jsonParse(APIGWEXPCONFIG)
                        echo "${lapigwconfig}"                             
                    }    

                    //**************************************************************************** 

                    // AMPLIFY
                    //**************************************************************************** 

                    sh'''
                        #!/bin/bash
                        dldir="./ampapp_functions"
                        [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                        [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    '''
                    sleep(time:3,unit:"SECONDS")

                    AMPAPPEXPORTFILE= sh(script: 'cat AmpFNameList.json', returnStdout: true).trim()
                    def ampparsedexpJson = new groovy.json.JsonSlurperClassic().parseText(AMPAPPEXPORTFILE)
                    for(int ampi = 0; ampi < ampparsedexpJson.size(); ampi++){
                        def APIGWCONFIGEXPNAME = sh(script: "aws amplify get-app --app-id ${ampparsedexpJson[ampi].ampId} --output json ${' > '}${'./'}${'ampapp_functions'}/${ampparsedexpJson[ampi].ampname}${'.json'}", returnStdout: true).trim()
                        echo"${APIGWCONFIGEXPNAME}"
                    }  
                    //**************************************************************************** 
                
                }
            }
        }
    }

    stage('Import Lambdas/S3/LEX/DynamoDB/APIGW/AMPLIFY - Target Origin'){
        steps {
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    
                    // Lambdas
                    //**************************************************************************** 

                     //sh'''
                     //   #!/bin/bash
                     //   dldir="./lam_functions"
                     //   [ "$dldir" == "" ] && { echo "Usage: $0 directory"; exit 1; }
                     //   [ -d "${dldir}" ] &&  echo "Directory $dldir Found." || mkdir -p "$dldir" echo "Directory $dldir Created."
                    //'''
                    
                    sleep(time:3,unit:"SECONDS")

                    //LAMBDAENVIMPORTFILE= sh(script: 'cat LamFNameList.json', returnStdout: true).trim()
                    //def impparsedJson = new groovy.json.JsonSlurperClassic().parseText(LAMBDAENVIMPORTFILE)
                    //for(int impi = 0; impi < impparsedJson.size(); impi++){
                    //    def impshret =  sh(script: "aws lambda create-function --function-name ${impparsedJson[impi].lamname} --runtime ${impparsedJson[impi].lruntime} --zip-file ${'fileb:///'}${WORKSPACE}/${'lam_functions'}/${impparsedJson[impi].lamname}${'.zip'} --handler ${impparsedJson[impi].lhandler} --role ${impparsedJson[impi].lrole} --description ${impparsedJson[impi].ldescp}  --timeout ${impparsedJson[impi].ltimeout} --memory-size ${impparsedJson[impi].lmemory} --environment ${toJSON(impparsedJson[impi].lenvvar)}  ", returnStdout: true).trim()
                    //    echo "${impshret}" 
                    //}  

                    //**************************************************************************** 
                    // s3 Buckets
                    //**************************************************************************** 
                    S3BENVIMPORTFILE= sh(script: 'cat S3bFNameList.json', returnStdout: true).trim()
                    def s3bparsedJson = new groovy.json.JsonSlurperClassic().parseText(S3BENVIMPORTFILE)
                    for(int s3bi = 0; s3bi < s3bparsedJson.size(); s3bi++){
                        def s3bshret =  sh(script: "aws s3api create-bucket --acl ${s3bparsedJson[s3bi].s3acl} --bucket ${s3bparsedJson[s3bi].s3name} --region ${s3bparsedJson[s3bi].s3region} --create-bucket-configuration LocationConstraint=${s3bparsedJson[s3bi].s3location}", returnStdout: true).trim()
                        echo "${s3bshret}" 
                        def s3bshretpubpolicy =  sh(script: "aws s3api put-public-access-block --bucket ${s3bparsedJson[s3bi].s3name} --public-access-block-configuration ${'BlockPublicAcls='}${s3bparsedJson[s3bi].BlockPublicAcls},${'IgnorePublicAcls='}${s3bparsedJson[s3bi].IgnorePublicAcls},${'BlockPublicPolicy='}${s3bparsedJson[s3bi].BlockPublicPolicy},${'RestrictPublicBuckets='}${s3bparsedJson[s3bi].RestrictPublicBuckets}", returnStdout: true).trim()
                        echo "${s3bshretpubpolicy}" 
                        //def s3bshretpolicy =  sh(script: "aws s3api put-bucket-policy --bucket ${s3bparsedJson[s3bi].s3name} --policy ${toJSON(s3bparsedJson[s3bi].s3policy)}", returnStdout: true).trim()
                        //echo "${s3bshretpolicy}" 
                    }    
                    //**************************************************************************** 

                    //LEX BOTS
                    //**************************************************************************** 
                    LEXENVIMPORTFILE= sh(script: 'cat LexFNameList.json', returnStdout: true).trim()
                    def lexparsedJson = new groovy.json.JsonSlurperClassic().parseText(LEXENVIMPORTFILE)
                    for(int lexi = 0; lexi < lexparsedJson.size(); lexi++){
                        def IMPORTLEX = sh(script: "aws lex-models start-import --payload ${'fileb:///'}${WORKSPACE}/${'lex_functions'}/${lexparsedJson[lexi].lexname}${'.zip'} --resource-type ${lexparsedJson[lexi].lextype} --merge-strategy ${lexparsedJson[lexi].lexoverride}", returnStdout: true).trim()
                        echo "${IMPORTLEX}"
                    }   
                    //**************************************************************************** 
                        //sleep(time:180,unit:"SECONDS")
                    //**************************************************************************** 

                    // KINESIS STREAMS
                    //**************************************************************************** 
                    KINESISSTREAMSIMPORTFILE= sh(script: 'cat KinFNameList.json', returnStdout: true).trim()
                    def kinparsedJson = new groovy.json.JsonSlurperClassic().parseText(KINESISSTREAMSIMPORTFILE)
                    for(int kini = 0; kini < kinparsedJson.size(); kini++){
                        def IMPORTKINESIS = sh(script: "aws kinesis create-stream --stream-name ${kinparsedJson[kini].kstname} --shard-count ${kinparsedJson[kini].kshcount}", returnStdout: true).trim()
                        //--stream-mode-details ${toJSON(kinparsedJson[kini].kssmode)} 
                        echo "${IMPORTKINESIS}"
                    }   
                    //**************************************************************************** 

                    // KINESIS FIREHOSE STREAMS
                    //**************************************************************************** 
                    KINFIREHOSEIMPORTFILE= sh(script: 'cat KfhFNameList.json', returnStdout: true).trim()
                    def kfhparsedJson = new groovy.json.JsonSlurperClassic().parseText(KINFIREHOSEIMPORTFILE)
                    for(int kfhi = 0; kfhi < kfhparsedJson.size(); kfhi++){
                        def IMPORTKINFIREHOSE = sh(script: "aws firehose create-delivery-stream --delivery-stream-name ${kfhparsedJson[kfhi].kfhname} --delivery-stream-type ${kfhparsedJson[kfhi].kfhstype} --s3-destination-configuration ${toJSON(kfhparsedJson[kfhi].kfhs3arnconfig)}", returnStdout: true).trim()
                        echo "${IMPORTKINFIREHOSE}"
                    }   

                    //**************************************************************************** 

                    // IMPORT API GW
                    //**************************************************************************** 
                    APIGWIMPORTFILE= sh(script: 'cat ApgFNameList.json', returnStdout: true).trim()
                    def apgparsedimpJson = new groovy.json.JsonSlurperClassic().parseText(APIGWIMPORTFILE)
                    for(int apgj = 0; apgj < apgparsedimpJson.size(); apgj++){

                    sh """

                    cd ${WORKSPACE}/${"apigw_funcitons/"}
                    sed s/${apgparsedimpJson[apgj].apiSstage}/${apgparsedimpJson[apgj].apiTstage}/${'g'} ${WORKSPACE}/${"apigw_funcitons/"}${apgparsedimpJson[apgj].apiname}${'.json'} >> ${'removeddev'}${apgparsedimpJson[apgj].apiname}${'.json'}
                    sed s/${apgparsedimpJson[apgj].apiSregion}/${apgparsedimpJson[apgj].apiTregion}/${'g'} ${WORKSPACE}/${"apigw_funcitons/"}${'removeddev'}${apgparsedimpJson[apgj].apiname}${'.json'} >> ${'removedregion'}${apgparsedimpJson[apgj].apiname}${'.json'}
                    sed s/${apgparsedimpJson[apgj].apiname}/${apgparsedimpJson[apgj].apiname}${'_prd'}/${'g'} ${WORKSPACE}/${"apigw_funcitons/"}${'removedregion'}${apgparsedimpJson[apgj].apiname}${'.json'} >> ${'removedtitle'}${apgparsedimpJson[apgj].apiname}${'.json'}
                    cp ${'removedtitle'}${apgparsedimpJson[apgj].apiname}${'.json'} ${apgparsedimpJson[apgj].apiname}${'.json'} 
                    rm ${'removeddev'}${apgparsedimpJson[apgj].apiname}${'.json'}
                    rm ${'removedregion'}${apgparsedimpJson[apgj].apiname}${'.json'}
                    rm ${'removedtitle'}${apgparsedimpJson[apgj].apiname}${'.json'}
            
                    """
                
                    //import                                
                    def APIGWIMPORTCONFIG = sh(script: "aws apigateway import-rest-api --cli-binary-format ${apgparsedimpJson[apgj].apiBformat} --body ${"file:///"}${WORKSPACE}/${"apigw_funcitons/"}${apgparsedimpJson[apgj].apiname}${'.json'}", returnStdout: true).trim()
                    def lapigwjsonconfig = jsonParse(APIGWIMPORTCONFIG)
                    echo "${lapigwjsonconfig}"

                    
                    sleep(time:10,unit:"SECONDS")
                    //deploy APIGW
                    def APIGWDEPLOYCONFIG = sh(script: "aws apigateway create-deployment --rest-api-id ${lapigwjsonconfig.id} --stage-name ${apgparsedimpJson[apgj].apiTstage} --stage-description ${apgparsedimpJson[apgj].apiTStagdescription} --description ${apgparsedimpJson[apgj].apiTGenldescription} ", returnStdout: true).trim()
                    def lapigwdeployconfig = jsonParse(APIGWDEPLOYCONFIG)
                    echo "${lapigwdeployconfig}"      
                    }    

                    //**************************************************************************** 

                    // IMPORT AMPLIFY
                    //**************************************************************************** 

                    AMPAPPIMPORTFILE= sh(script: 'cat AmpFNameList.json', returnStdout: true).trim()
                    def ampparsedimpJson = new groovy.json.JsonSlurperClassic().parseText(AMPAPPIMPORTFILE)

                    for(int ampj = 0; ampj < ampparsedimpJson.size(); ampj++){

                    sh """
                        cd ${WORKSPACE}/${"ampapp_functions/"}
                        sed s/${ampparsedimpJson[ampj].ampSstage}/${ampparsedimpJson[ampj].ampTsage}/${'g'} ${WORKSPACE}/${"ampapp_functions/"}${ampparsedimpJson[ampj].ampname}${'.json'} >> ${'removeddev'}${ampparsedimpJson[ampj].ampname}${'.json'}
                        sed s/${ampparsedimpJson[ampj].ampSregion}/${ampparsedimpJson[ampj].ampTregion}/${'g'} ${WORKSPACE}/${"ampapp_functions/"}${'removeddev'}${ampparsedimpJson[ampj].ampname}${'.json'} >> ${'removedregion'}${ampparsedimpJson[ampj].ampname}${'.json'}
                        sed s/${ampparsedimpJson[ampj].ampname}/${ampparsedimpJson[ampj].ampname}${'_prd'}/${'g'} ${WORKSPACE}/${"ampapp_functions/"}${'removedregion'}${ampparsedimpJson[ampj].ampname}${'.json'} >> ${'removedtitle'}${ampparsedimpJson[ampj].ampname}${'.json'}
                        cp ${'removedtitle'}${ampparsedimpJson[ampj].ampname}${'.json'} ${ampparsedimpJson[ampj].ampname}${'.json'} 
                        rm ${'removeddev'}${ampparsedimpJson[ampj].ampname}${'.json'}
                        rm ${'removedregion'}${ampparsedimpJson[ampj].ampname}${'.json'}
                        rm ${'removedtitle'}${ampparsedimpJson[ampj].ampname}${'.json'}
                    """

                    //Create Amplify App
                    def APIGWCONFIGEXPNAME = sh(script: "aws amplify create-app --name ${ampparsedimpJson[ampj].ampname} --description ${ampparsedimpJson[ampj].ampdepDes} --platform ${ampparsedimpJson[ampj].ampplatform} ", returnStdout: true).trim()
                    def ampparsedcatJson = jsonParse(APIGWCONFIGEXPNAME)
                    echo"${APIGWCONFIGEXPNAME}"

                    sleep(time:10,unit:"SECONDS")
                    // Create Branch Amplify Code
                    def APIGWCONFIGBRNNAME = sh(script: "aws amplify create-branch --app-id ${ampparsedcatJson['app'].appId} --branch-name ${ampparsedimpJson[ampj].ampdepBranch} --stage ${ampparsedimpJson[ampj].ampdepStage} --enable-auto-build", returnStdout: true).trim()
                    echo"${APIGWCONFIGBRNNAME}"

                    sleep(time:10,unit:"SECONDS")

                    // Pushing ZIP Code to Amplify Code
                    try{
                        def APIGWCONFIGDEPNAME = sh(script: "aws amplify start-deployment --app-id ${ampparsedcatJson['app'].appId} --branch-name ${ampparsedimpJson[ampj].ampdepBranch} --source-url ${ampparsedimpJson[ampj].ampsourceurl} ", returnStdout: true).trim()
                        echo"${APIGWCONFIGDEPNAME}"
                    }catch(Exception ex) {
                        println("Catching the exception");
                    }

                //**************************************************************************** 

                    } 
                
                }
            }
        }
    }
    

    stage('Collected all ARNs Lambdas/S3/LEX/DynamoDB/APIGW/AMPLIFY Build -Source Target'){
        steps {
            withAWS(credentials:'41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {

                    // Lamda Function Names
                    //LAMCONFIGFILE= sh(script: 'cat LamFNameList.json', returnStdout: true).trim()
                    //def lamconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(LAMCONFIGFILE)
                    //for(int lcfc = 0; lcfc < lamconfigfileObj.size(); lcfc++){
                    //    def LAMCONFIGFILERET = sh(script: "aws lambda get-function --function-name ${lamconfigfileObj[lcfc].lamname}", returnStdout: true).trim()
                    //    def lamarndetails = jsonParse(LAMCONFIGFILERET)
                    //    AWS_CONFIG_LAMCONFIGFILE = AWS_CONFIG_LAMCONFIGFILE + lamarndetails['Configuration']['FunctionArn'] + ','
                    //}    
                    echo"${AWS_CONFIG_LAMCONFIGFILE}"
                
                    // S3Bucket Configuration
                    S3BCONFIGFILE= sh(script: 'cat S3bFNameList.json', returnStdout: true).trim()
                    def s3bconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(S3BCONFIGFILE)
                    for(int s3bc = 0; s3bc < s3bconfigfileObj.size(); s3bc++){
                        if(s3bconfigfileObj[s3bc].useawsconnect == "true")
                        {
                                if(s3bconfigfileObj[s3bc].s3name =="amazon-connect-callrecording")
                                {
                                    AWS_CONFIG_S3BAWSCALLRECORD = "{\"StorageType\": \"S3\",\"S3Config\": {\"BucketName\": \"${s3bconfigfileObj[s3bc].s3name}\",\"BucketPrefix\": \"connect/Instance_Alias/CallRecordings\",\"EncryptionConfig\":{\"EncryptionType\":\"KMS\",\"KeyId\": \"arn:aws:kms:us-west-2:777025004749:key/34d3a798-db7d-4a90-9119-d21eaff653b5\"}}}"

                                }
                                if(s3bconfigfileObj[s3bc].s3name =="amazon-connect-chatrecording")
                                {
                                    AWS_CONFIG_S3BAWSCHATTRANSCRIPT = "{\"StorageType\": \"S3\",\"S3Config\": {\"BucketName\": \"${s3bconfigfileObj[s3bc].s3name}\",\"BucketPrefix\": \"connect/Instance_Alias/Chat_Transcripts\",\"EncryptionConfig\":{\"EncryptionType\":\"KMS\",\"KeyId\": \"arn:aws:kms:us-west-2:777025004749:key/34d3a798-db7d-4a90-9119-d21eaff653b5\"}}}"

                                }

                                if(s3bconfigfileObj[s3bc].s3name =="amazon-connect-exportreports")
                                {
                                    AWS_CONFIG_S3BAWSREPORTS = "{\"StorageType\": \"S3\",\"S3Config\": {\"BucketName\": \"${s3bconfigfileObj[s3bc].s3name}\",\"BucketPrefix\": \"connect/Instance_Alias/Reports\",\"EncryptionConfig\":{\"EncryptionType\":\"KMS\",\"KeyId\": \"arn:aws:kms:us-west-2:777025004749:key/34d3a798-db7d-4a90-9119-d21eaff653b5\"}}}"

                                }

                                if(s3bconfigfileObj[s3bc].s3name =="amazon-connect-agentsevents")
                                {
                                    AWS_CONFIG_S3BAWSAGENTEVENTS = "{\"StorageType\": \"S3\",\"S3Config\": {\"BucketName\": \"${s3bconfigfileObj[s3bc].s3name}\",\"BucketPrefix\": \"connect/Instance_Alias/AgentEvents\",\"EncryptionConfig\":{\"EncryptionType\":\"KMS\",\"KeyId\": \"arn:aws:kms:us-west-2:777025004749:key/34d3a798-db7d-4a90-9119-d21eaff653b5\"}}}"

                                }
                        }
                    }  
                    echo "${AWS_CONFIG_S3BAWSCALLRECORD}"  
                    echo "${AWS_CONFIG_S3BAWSCHATTRANSCRIPT}"  
                    echo "${AWS_CONFIG_S3BAWSREPORTS}"  
                    echo "${AWS_CONFIG_S3BAWSAGENTEVENTS}"  
                    
                    // Lex BOT Names
                    LEXCONFIGFILE= sh(script: 'cat LexFNameList.json', returnStdout: true).trim()
                    def lexconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(LEXCONFIGFILE)
                    for(int lexc = 0; lexc < lexconfigfileObj.size(); lexc++){
                        AWS_CONFIG_LEXCONFIGFILE = AWS_CONFIG_LEXCONFIGFILE + lexconfigfileObj[lexc].lexTregion +':'+ lexconfigfileObj[lexc].lexname + ','
                    }    
                    echo"${AWS_CONFIG_LEXCONFIGFILE}"


                    // Kinesis Stream Names
                    KINCONFIGFILE= sh(script: 'cat KinFNameList.json', returnStdout: true).trim()
                    def kinconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(KINCONFIGFILE)
                    for(int kinc = 0; kinc < kinconfigfileObj.size(); kinc++){
                        def KINCONFIGFILERET = sh(script: "aws kinesis describe-stream --stream-name ${kinconfigfileObj[kinc].kstname}", returnStdout: true).trim()
                        def kinarndetails = jsonParse(KINCONFIGFILERET)
                        AWS_CONFIG_KINCONFIGFILE = AWS_CONFIG_KINCONFIGFILE + kinarndetails['StreamDescription']['StreamARN'] 
                    }   
                    echo"${AWS_CONFIG_KINCONFIGFILE}" 
                    
                    // Kinesis Firehose Names
                    KFHCONFIGFILE= sh(script: 'cat KfhFNameList.json', returnStdout: true).trim()
                    def kfnconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(KFHCONFIGFILE)
                    for(int kfnc = 0; kfnc < kfnconfigfileObj.size(); kfnc++){
                        def KFNCONFIGFILERET = sh(script: "aws firehose describe-delivery-stream --delivery-stream-name ${kfnconfigfileObj[kfnc].kfhname}", returnStdout: true).trim()
                        def kfnarndetails = jsonParse(KFNCONFIGFILERET)
                        AWS_CONFIG_KFNCONFIGFILE = AWS_CONFIG_KFNCONFIGFILE + kfnarndetails['DeliveryStreamDescription']['DeliveryStreamARN'] 
                    }   
                    echo"${AWS_CONFIG_KFNCONFIGFILE}" 

                    // DynamoDB Table Names
                    DYBCONFIGFILE= sh(script: 'cat DybFNameList.json', returnStdout: true).trim()
                    def dybconfigfileObj = new groovy.json.JsonSlurperClassic().parseText(DYBCONFIGFILE)
                    for(int dybc = 0; dybc < dybconfigfileObj.size(); dybc++){
                        def DYBCONFIGFILERET = sh(script: "aws dynamodb describe-table --table-name ${dybconfigfileObj[dybc].dbtname}", returnStdout: true).trim()
                        def dybarndetails = jsonParse(DYBCONFIGFILERET)
                        AWS_CONFIG_DYBCONFIGFILE = AWS_CONFIG_DYBCONFIGFILE + dybarndetails['Table']['TableArn'] + ','
                    }   
                    echo"${AWS_CONFIG_DYBCONFIGFILE}" 
                }
            }
        }
    }

    stage('Create an Amazon Connect Instance - Target Origin'){
        steps {
            echo 'Creating the Amazon Connect Instance'
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                // List all the Buckets
                script {
                    String inboundCallsEnabled = "" 
                    String outboundCallsEnabled = ""
                    if(ENABLEINBOUNDCALLS.equals("true")) {
                        inboundCallsEnabled = "--inbound-calls-enabled" 
                    }
                    if(ENABLEOUTBOUNDCALLS.equals("true")) {
                        outboundCallsEnabled = "--outbound-calls-enabled" 
                    }
                    
                    def parsedJson =  sh(script: "aws connect create-instance --identity-management-type CONNECT_MANAGED --instance-alias ${INSTANCEALIAS} ${inboundCallsEnabled} ${outboundCallsEnabled}", returnStdout: true).trim()
                    echo "Instance details : ${parsedJson}"
                    def instance = jsonParse(parsedJson)
                    echo "ARN : ${instance.Arn}" 
                    ARN = instance.Arn
                }
            }
        }
    }
    stage('Check status of the Instance - Target Origin'){
        steps{
            echo 'Instance check'
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    echo "Waiting for the instance status to become Active withing 5 minutes "
                    def count = 1
                    while(count <= 60) {
                        println "Sleeping for 10 seconds, iteration ${count}"
                        sleep(time:10,unit:"SECONDS")
                        count++
                        def di =  sh(script: "aws connect describe-instance --instance-id ${ARN}", returnStdout: true).trim()
                        echo "Instance Describe : ${di}"
                        def instanceStatus = jsonParse(di)
                        String status = "ACTIVE"
                        echo "Status : ${instanceStatus.Instance.InstanceStatus.equals(status)}"
                        if(instanceStatus.Instance.InstanceStatus.equals(status)){
                            println("Instance creation is completed")
                            TRAGET_INSTANCE_ID = instanceStatus.Instance.Id
                            count = 61    
                        }
                        
                        sh"""
                        echo "${TRAGET_INSTANCE_ID}" > /var/lib/jenkins/newconnectInstance.txt
                        """
                    }
                }
            }
        }
    }
    
    stage('Approved Origins/Lambda/Lexbot/Contact-Flow/Contact-Trace-Records/Agent-Realtime-Events - Target Origin'){
        steps{
            echo 'Adding Origins/Lambda/Lexbot/Contact-Flow/Contact-Trace-Records/Agent-Realtime-Events'
            
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    // Approved Origins
                    def ao = APPROVEDORIGINS.split(",")
                    ao.each { obj ->
                        def di =  sh(script: "aws connect associate-approved-origin --instance-id ${ARN} --origin ${obj}", returnStdout: true).trim()
                        echo "Approved Origins : ${di}"
                    };
                    
                    // Lambdas 
                    def aola = AWS_CONFIG_LAMCONFIGFILE.split(",")
                    aola.each { objaola ->
                        def diobjaola =  sh(script: "aws connect associate-lambda-function --instance-id ${ARN} --function-arn ${objaola}", returnStdout: true).trim()
                        echo "Approved Lambdas : ${diobjaola}"
                    };
                    
                    // Lexbot
                    //def aolex = AWS_CONFIG_LEXCONFIGFILE.split(",")
                    //aolex.each { objaolex ->
                    //    String[] str = objaolex.split(":") 
                    //    def region = str[0]
                    //    def lexBot = str[1]
                    //    def diobjaolex =  sh(script: "aws connect associate-lex-bot --instance-id ${ARN} --lex-bot Name=${lexBot},LexRegion=${region}", returnStdout: true).trim()
                    //    echo "Instance LexBot : ${diobjaolex}"                            
                    //};
                    
                    // Contact Flow Logs
                    def dicfl =  sh(script: "aws connect update-instance-attribute --instance-id ${ARN} --attribute-type CONTACTFLOW_LOGS --value ${CONTACTFLOWLOGS}", returnStdout: true).trim()
                    echo "Enable Contact Flow Logs : ${dicfl}"
                    
                    // Contact Trace Records
                    def dictr =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CONTACT_TRACE_RECORDS --storage-config StorageType=KINESIS_FIREHOSE,KinesisFirehoseConfig={FirehoseArn=${AWS_CONFIG_KFNCONFIGFILE}}", returnStdout: true).trim()
                    echo "CTR : ${dictr}"

                    //Agent Realtime Events
                    def diare =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type AGENT_EVENTS --storage-config StorageType=KINESIS_STREAM,KinesisStreamConfig={StreamArn=${AWS_CONFIG_KINCONFIGFILE}}", returnStdout: true).trim()
                    echo "Agent Events : ${diare}"
                    
                }
            }
        }
    }
    
    stage('Enable Call Recordings - Target Origin'){
        steps{
            echo 'Enabling call recordings into S3'
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    String sc = AWS_CONFIG_S3BAWSCALLRECORD
                    sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                    echo sc
                    def js = jsonParse(sc)
                    sc = "StorageType=S3"
                    //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                    sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                    sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                    sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                    echo sc
                    js = null
                    def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CALL_RECORDINGS --storage-config ${sc}", returnStdout: true).trim()
                    echo "Call Recordings : ${di}"
                }
            }
        }
    }
    
    stage('Enable Chat Transcripts - Target Origin'){
        steps{
            echo 'Enabling chat transcripts into S3'
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    def sc = AWS_CONFIG_S3BAWSCHATTRANSCRIPT
                    sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                    echo sc
                    def js = jsonParse(sc)
                    sc = "StorageType=S3"
                    //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                    sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                    sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                    sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                    echo sc
                    js = null
                    def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CHAT_TRANSCRIPTS --storage-config ${sc}", returnStdout: true).trim()
                    echo "Chat Transcripts : ${di}"
                }
            }
        }
    }
    
    stage('Enable Scheduled Reports - Target Origin'){
        steps{
            echo 'Enabling S3 for storing scheduled reports'
            withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: params.TREGIONS) {
                script {
                    def sc = AWS_CONFIG_S3BAWSREPORTS
                    sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                    echo sc
                    def js = jsonParse(sc)
                    sc = "StorageType=S3"
                    //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                    sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                    sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                    sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                    echo sc
                    js = null
                    def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type SCHEDULED_REPORTS --storage-config ${sc}", returnStdout: true).trim()
                    echo "Chat Transcripts : ${di}"
                }
            }
        }
    }
    
    
    }
}
