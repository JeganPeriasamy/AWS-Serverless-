# AWS-Serverless-
# Serverless CICD Project for Contact Form
<img src="https://miro.medium.com/v2/resize:fit:1400/1*rucpayKCSLodSTWwNMWc4Q.png" alt="">


# Project Description:
  We are about to create a serverless CI/CD process for a serverless contact form. So whenever a user hits the website, the user can able to give their details and after clicking on submit the page will return  response as  “Thank you” . After completing the project we will do integration with some AWS services and achieve CI/CD for to enhance better automation and reduce repeated process and time consumption for developers.

# Prerequites:
For this project you need to have the following files
- /
  - website/
    - index.html
    - style.css
    - script.js
 - lambda/
    - index.js
    - package.json

Serverless.yaml -- for to create lambda, api gateway, dynamodb  [after serverless deploy serverless will create a .serverless folder and within that it will create two files -- cloudformation-createstack.json, cloudformation-updatestack.json] {you need to explain those two files }


# Procedure - Create an EC2 instance and integrate it with vs code and aws configure with IAM user 

# Install 
1. git
2. aws cli
3. sudo yum install nodejs
4. sudo npm install -g serverless

In the EC2 create a folder  - mkdir serverlesscicd
WE USE SAM in aws and WE USE SERVERELESS FRAMEWORK (3RD Party tool - To provision the services) 
•	vi serverless.yaml  - service: serverlesscicd

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x

# you can overwrite defaults here
  stage: prod
  region: eu-west-1
  

# functions:
  lambda:
    handler: index.handler ( OUR LAMDA CODE – WHAT GOING TO EXECUTE )
    event ( WE ARE GOING TO CREATE REST API ) 
      - http: ( REST API)
          path: /submit
          method: POST ( 2 METHODS – GET( Get lambda invoke) AND POST ) 


# create index.js  (This is the lambda code )

# vi Index.js

const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();
const { v4: uuidv4 } = require('uuid');

exports.handler = async (event) => {
  try {
    console.log('Raw input data:', event); // Add this line to log the raw input data

    const formData = {
      name: event.name,
      email: event.email,
      subject: event.subject,
      message: event.message,
    };

    const item = {
      SubmissionId: generateUUID(), // Generate a UUID
      ...formData, // Use the form data as attributes
    };

    // Store the form data in DynamoDB
    await storeFormData(item);

    return {
      statusCode: 200,
      body: JSON.stringify({ message: 'Form submitted successfully and this is serverless cicd v1' }),
    };
  } catch (error) {
    console.error(error);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Error submitting the form' }),
    };
  }
};

// here we will provide the dynamo db table name 
async function storeFormData(item) {
  const params = {
    TableName: 'ContactFormEntries',
    Item: item,
  };

  await dynamodb.put(params).promise();
}

function generateUUID() {
  return uuidv4();
}


	Run npm init --- > it will create a package.json – We are initiate the node js ( we user this file to deploy – we command deploy ,it searches the package.json and deploy it , suppose it is not there it fails )
 

	As it is a serverless proj we require aws dependencies , I’ve added this one line of code by seeing the error in the lambda console , add one more line in the package.json 
AWS dependencies
1.	use module to use other services 
2.	

 
	Here the index.js is the main lambda code to be executed in simple it is the name of the code , if you are changing the code or you want to run another code means you need to change the handler name for example in the serverless.yaml you need to provide the handler the code name here if the code name is testservice then you need to give as testservice.handler in the serverless.yaml 

	Run --- > npm install --- > it will create a node modules folder and package-lock.json will be created (For what it is created? = 
	Run -- > serverless deploy 
	By default It will create a lambda , rest api gateway based on the service+stage name+ function  name – in the serverlessyaml file and also api gateway stage name plus service name 
	If you again run the deploy command with the change in stage and lambda name it willcreate new lambda and api gateway 
	The serverless deploy command will package all the code in to the lambda and it will create a .serverless folder within that two cloud formation files will be there , cloudformation-create-stack.json
	cloudformation-update-stack.json,,, by default the serverless deploy command will upload our index.js code to s3 by as it creates two cloud formation files one to create s3 bucket and value of the output , that bucket will be imported in another update stack , and on this bucket only our index.js code and all our code will be uploaded , on the template you can find the s3 key , in that location the code will be there

 
 


 



In serverless.yaml 
 
	Even though we are giving http it will create rest api only , that is the latest update if you give rest instead of http , still it would rest api gateway but it will be older version of the rest api.
	So know about how to create http 

Create events in lambda 
{
  "name": "test",
  "email": "abd@example.com",
  "subject": "This is you",
  "message": "A message from you"
}
 

And click on save 










	Goto dynamo db and create a table as given in the same name in index.js 
 

	Edit the existing lambda role to have access to dynamo db , if you didn’t attach the role then the permission error will be visible in cloud  watch logs 
	Under configuration -- > permission --- > execution role -- > add permission -- > attach policy --- > dynamo db full access/ required access.
 

	Now run the test event, you should see  

	In the dynamodb you should see the content

 


	Go to api gateway
	Click on Post--- > Integration request
   





	Uncheck the lambda proxy integration
 










	Scroll down and click on mapping template and verify like configured below
 






	Create an s3 bucket for website hosting and make the bucket public
 
 
 
by using policy generator

	Enable website hosting with index.html
 
	Copy the s3website link 
 
	Go to api gateway click on submit resource 
 


	Actions -- > enable cors
	On that Allow access origin paste the s3 static website url
 
	It would throw a error 
 
	Now click on options--- > integration request
 


	Configure like below
 
	Click on POST---- > method response
 
	Type 200
 

	Now again enable CORS
 
	Now Deploy API under prod stage and copy the api url 
	Goto resources -- > test 
	Under request body 
 


	You should see the below response
 

	In dynamo db you should see the below response
 




	In the script.js give the api gateway url 
 







	Go to s3 and upload the below files 
 

	In s3 paste the following cors under cors configuration
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "POST"
        ],
        "AllowedOrigins": [
            "your-static-website-url"
        ],
        "ExposeHeaders": []
    }
]


	Hit the S3 website url under properties
	You should see the below success response
 

 





	In dynamo db you should see the below response
 
	Thus we have integrated Lambda-- > Api gateway -- > dynamodb -- > s3
	Now we are about to initiate CI/CD 
	Initially we need to integrate our working repo with github 
	Create a new repository in git hub 
 

 
	cd serverelesscidcd/
	ll
	Initiate Git --- > git init --- in the working repository
	git add*
	git config username and email
	git commit –m “first commit”
	git remote add abd https://github.com/abdulrahman0108/serverlesiccd.git

	run -- > ssh-keygen -t ed25519 -C your_email@example.com

	Press two enter 
	cd 
	cd .ssh
	ll
	it displays the all keys
	cat id_pub – copy the key
	Run -- > eval "$(ssh-agent -s)"
	 
	ssh-add ~/.ssh/id_ed25519
	Go to Git hub and add the public key in the ssh keys and add it 
	ssh –v git@github.com
	hi 7598273777 – it shows 
 



 



	for more clarity on above github steps follow the url 
  [ https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux ]
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account   ]

	Now we have created the ssh public key, now we need to add the public key in github
	Goto settings in github
 

	Click on add ssh and gpg keys
 
	Copy the public key from .ssh/
	Run -- > cat id_ed25519.pub
	Copy the ssh and paste on the github ssh key 
 
Now run -- > ssh -v git@github.com ,, you should see
 

	Again go to github clikc on settings -- > developer settings-- > generate classic tokens
 
	Give all the access and generate token and set expiration as never expire and copy the token id 
 
	Run -- > git push -u jegan master
	username : jegan
	Password : Token of git 
































	You should able to push from local to github
 
	Now go to code build 
	Build the project
 
	Save Token
 
















	Give the repository url
 
	Attach the existing role or create the role with the following permissions
 
Docker images provided by CodeBuild - AWS CodeBuild (amazon.com)
	Configure the environment like below 
 












	Cause the code build to avoid permission issues attach the following permission
 


	Give the buildspec.yaml name 
 

















	Under additional configuration
 











	Set the stage variable as given below
 
	Export the logs to cloudwatch if require s3 also and create build project
 
	Start the build and check for the logs
 

	The build will be expected to be failure cause we haven’t push the buildspec.yaml 
	Create buildspec.yaml in the working repository inside the project folder 

version: 0.2
run-as: root

phases:
   install:
       runtime-versions:
          nodejs: 18
       commands:
          - npm install -g serverless
          - npm install
   build:
       commands:
          - serverless deploy --stage ${STAGE_NAME}
 
cache:
   paths:
      - node_modules




	Push the buildspec.yaml to github
 

Retry the build 
If error – Give the Roles full Admin Access
 

	You should the build succeeded state along with the api gateway url
	BuiLd gives the POST – url and it cant be browsed 
 
	Now integrate the project with code pipeline
	Create the code pipeline like below
 
	Choose the source provider as github v1  and connect to github
 
Correction : Give webhook not aws codepipeline 
	Give a connection name 
	Click authorize aws-codesuite
	It will prompt to authorize – click on authorize
 














	Choose your repository with master branch
 
 
	Give the build provider name 
 
	As we have given the deploy in the buildspec yaml itself we can skip the integration with code deploy
 
	Create pipeline
 
	After creating the pipeline , the pipeline will run a dry build.
 

	Thus we have successfully integrated the project with lambda, api gateway, dynamodb,s3, github,codebuild, code pipeline
	Open the index.js in the local repository and edit the lambda code 
 
	When you push the changes
 








	The build will be started again
 
	Go to lambda and edit the saved test event
 
 
	Test it 
	Here the lambda response  code is changed to v6
 

	Thus if any changes in github repository will identified by githook in the code pipeline and it trigger the code build to build the project from the github repository and deploy the changes in the code. Thus we have achieved CI/CD integration.



Interview 
Frame the questions with integrating with services 
1.	ECS -  deploy and usage , procedure
2.	IAM Roles 
3.	List the aws services u know 
4.	say something about hobby and yourself
5.	How do you know about AWS and why do u choose it
6.	How to intregrate ECS and DynamoDb - 
7.	Aws SDK 
8.	What are services and Database U worked with list ? ---? ?
9.	How to install postgre in EC2 ( Browse)
10.	How to change website http  to https
11.	400 – Client requect error
12.	503 error – file not inside so check the file/permission/handler
13.	200 – success
