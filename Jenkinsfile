pipeline { 
    agent any 
    
   
    stages {
        stage('Delete-Lambdas') { 
            steps { 
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {
                script{
                    try{
                    sh'''
                    #!/bin/bash
                        download_code () {
                            local OUTPUT=$1
                            aws lambda  delete-function --function-name $OUTPUT
                        }
                        for run in $(aws lambda list-functions  --query 'Functions[].FunctionName' --output text);
                        do
                            download_code "$run"
                            echo $run
                        done
                    ''' 
                    }catch(Exception ex) {
                        println("Catching the exception");
                    }

                 }
               }
            }
        }
        stage('Delete-DeliveryStreamNames'){
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') {
                script{
                try{
                sh'''
                    #!/bin/bash
                        download_code () {
                            local OUTPUT=$1
                            aws firehose delete-delivery-stream --delivery-stream-name $OUTPUT --allow-force-delete
                        }
                        for run in $(aws firehose list-delivery-streams  --query 'DeliveryStreamNames[]' --output text);
                        do
                            download_code "$run"
                            echo $run
                        done
                    '''
                }catch(Exception ex) {
                        println("Catching the exception");
                    }
                }
                }

                
            }
        }
        stage('Delete-StreamNames') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                script{

                    try{
                    sh'''
                    #!/bin/bash
                        download_code () {
                            local OUTPUT=$1
                            aws kinesis delete-stream --stream-name $OUTPUT --enforce-consumer-deletion
                        }
                        for run in $(aws kinesis list-streams --query 'StreamNames[]' --output text);
                        do
                            download_code "$run"
                            echo $run
                        done
                    '''
                    }catch(Exception ex) {
                        println("Catching the exception");
                    }
                }
               }
            }
        }
        
        stage('Delete-Amplify') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{

                        try{
                        sh'''
                        #!/bin/bash
                            download_code () {
                                local OUTPUT=$1
                                aws amplify delete-app --app-id $OUTPUT
                            }
                            for run in $(aws amplify list-apps --query 'apps[].appId' --output text);
                            do
                                download_code "$run"
                                echo $run
                            done
                        '''
                        }catch(Exception ex) {
                        println("Catching the exception");
                    }
                    }
               }
            }
        }


        stage('Delete-DynamoDB') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{

                        try{
                        sh'''
                        #!/bin/bash
                            download_code () {
                                local OUTPUT=$1
                                aws dynamodb delete-table --table-name $OUTPUT
                            }
                            for run in $(aws dynamodb list-tables --query 'TableNames[]' --output text);
                            do
                                download_code "$run"
                                echo $run
                            done
                        '''
                        }catch(Exception ex) {
                        println("Catching the exception");
                    }
                    }
               }
            }
        }
        
        stage('Delete-S3Buckets') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{

                        try {
                        sh'''
                        #!/bin/bash
                            aws s3api delete-bucket --bucket "amazon-connect-callrecording" --region us-west-2
                            aws s3api delete-bucket --bucket "amazon-connect-chatrecording" --region us-west-2
                            aws s3api delete-bucket --bucket "amazon-connect-exportreports" --region us-west-2
                            aws s3api delete-bucket --bucket "amazon-connect-agentsevents" --region us-west-2
                        '''

                        }catch(Exception ex) {
                        println("Catching the exception");
                    }
                    }
               }
            }
        }

        stage('Delete-LEX') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{
                        try{
                        sh'''
                        #!/bin/bash
                            aws  lex-models  delete-bot --name "BookTrip_enGB"
                        '''
                        }catch(Exception ex) {
                        println("Catching the exception");
                    }
                    }
               }
            }
        }
        
        stage('Delete-APIGW') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{
                        
                        try{
                        sh'''
                        #!/bin/bash
                            download_code () {
                                local OUTPUT=$1
                                aws apigateway delete-rest-api --rest-api-id $OUTPUT
                            }
                            for run in $(aws apigateway get-rest-apis --query 'items[].id' --output text);
                            do
                                download_code "$run"
                                echo $run
                            done
                        '''   
                        }catch(Exception ex) {
                        println("Catching the exception");
                    }                     
                    }
               }
            }
        }

        stage('Delete-AWSInstance') {
            steps {
                withAWS(credentials: '41adfa6b-ece9-44a7-8ae5-e605f2898560', region: 'us-west-2') { 
                    script{
                        try{
                        
                        sh'''
                        #!/bin/bash
                            download_code () {
                                local OUTPUT=$1
                                aws connect delete-instance --instance-id $OUTPUT
                            }
                            for run in $(aws connect list-instances --query 'InstanceSummaryList[].Id' --output text);
                            do
                                download_code "$run"
                                echo $run
                            done
                        ''' 
                        }catch(Exception ex) {
                        println("Catching the exception");
                    }                       
                    }
               }
            }
        }

    }
}
