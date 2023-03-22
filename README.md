The auto_alarms deployment utilizes three separate CloudFormation stacks. 

To create the SNS topic, deploy the CloudFormation stack in the appropriate account and region: 

Download the file: CloudWatchAutoAlarms-SNS.yaml 

Deploy the CloudWatchAutoAlarms-SNS stack: 
aws cloudformation create-stack --stack-name amazon-cloudwatch-auto-alarms-sns-topic --template-body file://CloudWatchAutoAlarms-SNS.yaml --parameters ParameterKey=OrganizationID,ParameterValue="" --region us-west-2 

Parameters of the CloudWatchAutoAlarms-SNS stack: 
	OrganizationID 
The stack may be deployed to an OU ID from AWS Organizations. For a single account deployment, leave this parameter blank 

Retrieve the CloudWatchAutoAlarms-SNS stack details:
aws cloudformation describe-stacks --stack-name amazon-cloudwatch-auto-alarms-sns-topic --query "Stacks[0].Outputs[?ExportName=='amazon-cloudwatch-auto-alarms-sns-topic-arn'].OutputValue" --output text --region us-west-2 


To Create an S3 bucket in the appropriate account and region to house the Lambda function for the auto-alarms stack: 

Download the file: CloudWatchAutoAlarms-S3.yaml 

Deploy the CloudWatchAutoAlarms-S3 stack: 
aws cloudformation create-stack --stack-name cloudwatch-auto-alarms-s3-bucket --template-body file://CloudWatchAutoAlarms-S3.yaml --parameters ParameterKey=OrganizationID,ParameterValue="" --region us-west-2 

Retrieve the CloudWatchAutoAlarms-S3 stack details: 
aws cloudformation describe-stacks --stack-name cloudwatch-auto-alarms-s3-bucket --query "Stacks[0].Outputs[?ExportName=='amazon-cloudwatch-auto-alarms-bucket-name'].OutputValue" --output text --region us-west-2 


To create the Cloudwatch AutoAlarms:

Download the files: actions.py and cw_auto_alarms.py 
Zip the two python files together in one folder.

Copy the files to the created S3 bucket:
aws s3 cp S3DeploymentKey.zip s3://<BuckName> 
For S3DeploymentKey enter the name of the zip file. This will be used again during the CloudWatchAutoAlamrs deployment 
For <BuckName>, enter the name of the s3 bucket returned above. 

Download the file: CloudWatchAutoAlarms.yaml 

Deploy the stack: 
aws cloudformation create-stack --stack-name amazon-cloudwatch-auto-alarms --template-body file://CloudWatchAutoAlarms.yaml --capabilities CAPABILITY_IAM --parameters ParameterKey=S3DeploymentKey,ParameterValue=S3DeploymentKey ParameterKey=S3DeploymentBucket,ParameterValue=S3DeploymentBucket ParameterKey=AlarmNotificationARN,ParameterValue=AlarmNotificationARN 

Deploy the stack with the following parameters: 
	AlarmNotificationARN – The ARN of the SNS Topic created previously with the CloudWatchAutoAlarms-SNS stack. 
	S3DeploymentBucket – The name of the S3 bucket created that contains the lambda functions .zip file 
	S3DeploymentKey -	The name of the zip file in the S3DeploymentBucket
	(Optional)EventState – ENABLED 
	(Optional) Memory – 128 	

After deployment is complete, all instances that enter the running state with the tag: Create_Auto_Alarms the alarms will be created automatically.  
