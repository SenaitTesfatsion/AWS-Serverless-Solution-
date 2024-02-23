# Building A Serverless Solution In The AWS Cloud
This project presents the development of a serverless solution on the AWS cloud tailored for a web backend with decoupled application components. The architecture is designed to seamlessly accommodate fluctuating demands by dynamically scaling in and out as necessary.

![serverless](https://user-images.githubusercontent.com/110143245/222287619-b02ab3d3-9c14-4a86-953c-c6eac72a84e7.png)

# Cloud Tools Used;
- _AWS Identity and Access Management (IAM) policy and user_
- _DynamoDB table_
- _Lambda functions_
- _Simple Queue Service (Amazon SQS) queue_
- _Simple Notification Service (Amazon SNS) topic_
- _API Gateway_
- _CloudWatch Logs_


# Instructions

* [Create IAM Policies](create-iam-policies)
* [Create IAM Roles and attach policies to the roles](create-iam-roles-and-attach-policies-to-the-roles)
* [Create A DynamoDB table](create-a-dynamodb-table)
* [Create an SQS queue](create-an-sqs-queue)
* [Create a Lambda function and set up triggers](create-a-lambda-function-and-set-up-triggers)
* [Enable DynamoDB Streams](enable-a-dynamodb-streams)
* [Create an SNS topic and set up subscriptions](create-an-sns-topic-and-set-up-subscriptions)
* [Create an AWS Lambda function to publish a message to the SNS topic](create-an-aws-lambda-function-to-publish-a-message-to-the-sns-topic)
* [Create an API with Amazon API Gateway](create-an-api-with-amazon-api-gateway)
* [Test the Architecture by using API Gateway](test-the-architecture-by-using-api-gateway)
* [Cleaning UP](cleaning-up)


# Stage 1 – Create IAM Policies 
### Stage 1A – Create IAM Policy for the Dynamodb table
- Sign in to the AWS management console
- Select the us-east-1 region
- Select **IAM** from the AWS services
- In the navigation pane, choose **Policies**
- Choose **Create policy** 
- In the **JSON tab**, paste the following code:
~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:DescribeTable"
            ],
            "Resource": "*"
        }
    ]
}
~~~
- Choose **Next: Tags** and then choose **Next: Review**

- For the policy **name**, enter "Lambda-Write-DynamoDB"

- Choose **Create policy**

_This JSON script grants permissions to put items into the DynamoDB table._


### Stage 1B - Create IAM Policy for SNS 

- Select **IAM** from the AWS services
- In the navigation pane, choose **Policies**
- Choose **Create policy** 
- In the **JSON tab**, paste the following code:
~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sns:Publish",
                "sns:GetTopicAttributes",
                    "sns:ListTopics"
            ],
                "Resource": "*"
        }
    ]
 }
 ~~~

- Choose **Next: Tags** and then choose **Next: Review**

- For the policy **name**, enter "Lambda-SNS-Publish"

- Choose **Create policy**

_This policy grants SNS to get, list, and publish topics that are received by Lambda._

### Stage 1C- Create IAM Policy for Lambda

- Select **IAM** from the AWS services
- In the navigation pane, choose **Policies**
- Choose **Create policy** 
- In the **JSON tab**, paste the following code:
~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetShardIterator",
                "dynamodb:DescribeStream",
                "dynamodb:ListStreams",
                "dynamodb:GetRecords"
            ],
            "Resource": "*"
        }
    ]
}
~~~
- Choose **Next: Tags** and then choose **Next: Review**

- For the policy **name**, enter "Lambda-DynamoDBStreams-Read"

- Choose **Create policy**

_This policy allows lambda to get records from DynamoDB Streams._

### Stage 1D- Create IAM Policy for SQS
- Select **IAM** from the AWS services
- In the navigation pane, choose **Policies**
- Choose **Create policy** 
- In the **JSON tab**, paste the following code:
~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sqs:DeleteMessage",
                "sqs:ReceiveMessage",
                "sqs:GetQueueAttributes",
                "sqs:ChangeMessageVisibility"
            ],
            "Resource": "*"
        }
    ]
}
~~~

- Choose **Next: Tags** and then choose **Next: Review**

- For the policy **name**, enter "Lambda-SQS-Read"

- Choose **Create policy**

_This policy allows lambda to read messages that placed in Amazon SQS._

# Stage 2 - Create IAM Roles and attach policies to the roles
### Stage 2A- Create IAM role for DynamoDB
- In the navigation pane of the **IAM** dashboard, choose **Roles**
- Choose **Create role** and in the **Select trusted entity** page, configure the following settings:
- **Trusted entity type**: AWS service
- **Common use cases**: Lambda
- Choose **Next**
- On the **Add permissions** page, select **Lambda-Write-DynamoDB** and **Lambda-Read-SQS**
- Choose **Next**
- For Role **name**, enter "Lambda-SQS-DynamoDB"
- Choose **Create role**

### Stage 2B - Create IAM role for Lambda
- In the navigation pane of the **IAM** dashboard, choose **Roles**
- Choose **Create role** and in the **Select trusted entity** page, configure the following settings:
- **Trusted entity type**: AWS service
- **Common use cases**: Lambda
- Choose **Next**
- On the **Add permissions** page, select **Lambda-SNS-Publish** and **Lambda-DynamoDBStreams-Read**
- Choose **Next**
- For Role **name**, enter "Lambda-DynamoDBStreams-read"
- Choose **Create role**
_This role grants permissions to obtain records from the DynamoDB streams and send the records to Amazon SNS._

### Stage 2C - Create IAM role for API Gateway
- In the navigation pane of the **IAM** dashboard, choose **Roles**
- Choose **Create role** and in the **Select trusted entity** page, configure the following settings:
- **Trusted entity type**: AWS service
- **Common use cases**: API Gateway
- Choose **Next**
- On the **Add permissions** page, select **AmazonAPIGatewayPushToCloudWatchLogs**
- Choose **Next**
- For Role **name**, enter "APIGateway-SQS"
- Choose **Create role**
_This role grants permissions to send data to the SQS queue and push logs to Amazon CloudWatch for troubleshooting._

# Stage 3 - Create A DynamoDB table
- In the search box of the AWS services, enter **DynamoDB**
- From the list, choose the **DynamoDB service**
- On the **Get started** card, choose **Create table** and configure the following settings:
- **Table**: orders
- **Partition key**: orderID
- **Data type**: Keep String
- Leave everything else as default, and choose **Create table**

_This DynamoDB table ingests data that's passed on through API Gateway._

# Stage 4 - Create an SQS queue
- In the search box of the AWS services, enter **SQS**
- On the **Get started** card, choose **Create queue**
- Configure the following settings:
- **Name**: POC-Queue
- **Access Policy**: Basic
- **Define who can send messages to the queue**: Select Only the specified AWS accounts, IAM users and roles
- In the box for this option, **paste the Amazon Resource Name (ARN)** for the APIGateway-SQS IAM role
- **Define who can receive messages from the queue**: Select Only the specified AWS accounts, IAM users and roles.
- In the box for this option, **paste the ARN for the Lambda-SQS-DynamoDB IAM role**
- Choose **Create queue**

_This Amazon SQS receives data records from API Gateway, stores them, and then sends them to a database._

# Stage 5 - Create a Lambda function and set up triggers
 - In the search box of the AWS services, enter **Lambda**
 - Choose **Create function** and configure the following settings:
- **Function option**: Author from scratch
- **Function name**: POC-Lambda-1
- **Runtime**: Python 3.9
- **Change default execution role**: Use an existing role
- **Existing role**: Lambda-SQS-DynamoDB
- Choose **Create function**

_This Lambda function reads messages from the SQS queue and writes an order record to the DynamoDB table._

### Stage 5A - Set up Amazon SQS as a trigger to invoke the function
- Expand the **Function overview** section
- Choose **Add trigger**
- For **Trigger configuration**, enter **SQS** and choose the service in the list
- For **SQS queue**,** choose "POC-Queue"
- Add the trigger by choosing **Add**

### Stage 5B - Add and deploy the function code
- On the **POC-Lambda-1** page, in the **Code** tab, replace the default Lambda function code with the following code:
~~~
import boto3, uuid

client = boto3.resource('dynamodb')
table = client.Table("orders")

def lambda_handler(event, context):
    for record in event['Records']:
        print("test")
        payload = record["body"]
        print(str(payload))
        table.put_item(Item= {'orderID': str(uuid.uuid4()),'order':  payload})
~~~
- Choose **Deploy**
_The Lambda code passes arguments to a function call._

### Stage 5C -Test the POC-Lambda-1 Lambda function 
- In the **Test** tab, create a new event that has the following settings:
- **Event name**: POC-Lambda-Test-1
- **Template-Optional**: SQS
- The SQS template appears in the **Event JSON** field
- Save your changes and choose **Test**

_After the Lambda function runs successfully, the “Execution result: succeeded” message appears in the notification banner in the **Test** section._

### Stage 5D - Verify that the Lambda function adds the test message to a database
- In the search box of the AWS services, enter **DynamoDB**
- In the navigation pane, choose **Explore items**
- Select the **orders** database. Under **Items returned**, the **orders** table returns “Hello from SQS!” from the Lambda function test.

 # Stage 6 - Enable DynamoDB Streams

- In the DynamoDB console, in the **Tables** section of the navigation pane, choose **Update settings**
- In the **Tables** card, Select **orders** 
- Choose the **Exports and streams** tab
- In the **DynamoDB stream details** section, choose **Enable**
- For **View type**, choose **New image**
- Choose **Enable stream**

_The DynamoDB stream captures information about every modification to data items in the table._

# Stage 7 - Create an SNS topic and set up subscriptions

### Step 7A - Create a topic in the notification service
 - In the search box of the AWS services, enter **SNS**
- On the **Create topic** card, enter "POC-Topic" and choose **Next step**
- In the **Details** section, keep the **Standard** topic type selected and choose **Create topic**
- On the **POC-Topic** page, copy the ARN of the topic that you just created and save it for your reference

### Stage 7B - Subscribe to email notifications
- On the **Subscriptions** tab, choose **Create subscription**
- For **Topic ARN**, choose the "ARN for POC-Topic"
- For **Protocol**, choose **Email**
- For **Endpoint**, enter your email address
- Choose **Create subscription**
- The confirmation message is sent to the specified email address and after you receive the confirmation email message, confirm the subscription. 

# Stage 8 - Create an AWS Lambda function to publish a message to the SNS topic

### Step 8A - Create a POC-Lambda-2 function
- In the AWS Management Console, search for and open **AWS Lambda**
- Choose **Create function**, and configure the following settings:
- **Function option**: Author from scratch
- **Function name**: POC-Lambda-2
- **Runtime**: Python 3.9
- **Change default execution role**: Use an existing role
- **Existing role**: Lambda-DynamoDBStreams-SNS
- Choose **Create function**
_This role grants permissions to get records from DynamoDB Streams and send them to Amazon SNS._

### Stage 8B - Set up DynamoDB as a trigger to invoke a Lambda function
- In the **Function overview** section, choose **Add trigger** and configure the following settings:
- **Trigger configuration**: Enter DynamoDB and from the list, Choose **DynamoDB**
- **DynamoDB table**: orders
- Leave everything else as default and choose **Add**
- In the **Configuration** tab, Choose **Triggers** section and "Enable DynamoDB state"

### Stage 8C - Configure the second Lambda function
- Choose the **Code** tab and replace the Lambda function code with the following code:
~~~
import boto3, json

client = boto3.client('sns')

def lambda_handler(event, context):

    for record in event["Records"]:

        if record['eventName'] == 'INSERT':
            new_record = record['dynamodb']['NewImage']    
            response = client.publish(
                TargetArn='<Enter Amazon SNS ARN for the POC-Topic>',
                Message=json.dumps({'default': json.dumps(new_record)}),
                MessageStructure='json'
            )
~~~
_In the function code, replace the TargetArn value with the ARN for the Amazon SNS POC-Topic. Make sure that you remove the placeholder angle brackets (<>)._
- Choose **Deploy**

### Stage 8D - Test the POC-Lambda-2 Lambda function
- On the **Test** tab, create a new event and for **Event name**, enter "POC-Lambda-Test-2"
- For **Template-optional**, enter "DynamoDB" and from the list, choose **DynamoDB-Update**
- Save your changes and choose **Test**

_After the Lambda function successfully runs, the “Execution result: succeeded” message should appear in the notification banner in the **Test** section._

_After you receive the email message to the specified email address, confirm the subscription._


# Stage 9 - Create an API with Amazon API Gateway
- In the AWS Management Console, search for and open **API Gateway**
- On the **REST API** card with a public authentication, choose **Build** and configure the following settings:
- **Choose the protocol**: REST
- **Create new API**: New API
- **API name**: POC-API
- **Endpoint Type**: Regional
- Choose **Create API**
- On the **Actions** menu, choose **Create Method**
- Open the method menu by choosing the down arrow, and choose ****POST**. Save your changes by choosing the check mark.
- In the **POST - Setup** pane, configure the following settings:
- **Integration type**: AWS Service
- **AWS Region**: us-east-1
- **AWS Service**: Simple Queue Service (SQS)
- **AWS Subdomain**: Keep empty
- **HTTP method**: POST
- **Action Type**: Use path override
- **Path override**: Enter your account ID followed by a slash (/) and the name of the POC queue
- **Execution role**: Paste the ARN of the APIGateway-SQS role
- **Content Handling**: Passthrough
- Save your changes
- Choose the **Integration Request** card
- Scroll to the bottom of the page and expand **HTTP Headers**
- **Choose Add header**
- For **Name**, enter "Content-Type"
- For **Mapped from**, enter 'application/x-www-form-urlencoded'
- Save your changes to the **HTTP Headers**section by choosing the check mark.
- Expand **Mapping Templates** and for **Request body passthrough**, choose **Never**
- Choose **Add mapping template** and for **Content-Type**, enter "application/json"
- Save your changes by choosing the check mark.
- For **Generate template**, enter the following command: Action=SendMessage&MessageBody=$input.body instead of the default template
- Choose **Save**

# Stage 10 - Test the Architecture by using API Gateway
- In the **API Gateway** console, return to the **POST - Method Execution** page and choose **Test**
- In the **Request Body** box, enter:
~~~
{  "item": "latex gloves",
"customerID":"12345"}
~~~
- Choose **Test**
- The “Successfully completed execution” message with the 200 response in the logs on the right should appear and you will receive an email notification with the new entry.

# Stage 11 - Cleaning Up

### A. Delete the DynamoDB table
- Open the DynamoDB console
- In the navigation pane, choose **Tables**
- Select the **orders** table
- Choose **Delete** and confirm your actions
### B. Delete the Lambda functions
-  Open the Lambda console
- Select the Lambda functions created in this project: **POC-Lambda-1** and **POC-Lambda-2**
- Choose **Actions, Delete**
- Confirm your actions and close the dialog box
### C. Delete the SQS queue
- Open the Amazon SQS console
- Select the queue created in this project
- Choose Delete and confirm your actions
### D. Delete the SNS topic and subscriptions
- Open the Amazon SNS console
- In the navigation pane, choose **Topics**
- Select **POC-Topic**
- Choose **Delete** and confirm your actions
- In the navigation pane, choose **Subscriptions**
- Select the subscription created in this project and choose **Delete**
Confirm your actions
### E. Delete the API Gateway
- Open the API Gateway console
- Select **POC-API**
- Choose **Actions, Delete**
- Confirm your actions
### F. Delete the IAM roles and policies
- Open the IAM console
- In the navigation pane, choose **Roles**
- Delete the following roles and confirm your actions:
- **APIGateway-SQS**
- **Lambda-SQS-DynamoDB**
- **Lambda-DynamoDBStreams-SNS**
- In the navigation pane, choose **Policies**
- Delete the following custom policies and confirm your actions:
-  **Lambda-DynamoDBStreams-Read**
-  **Lambda-SNS-Publish**
-  **Lambda-Write-DynamoDB**
-  **Lambda-Read-SQS**




