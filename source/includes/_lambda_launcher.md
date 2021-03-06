# The Backand Lambda Launcher

The Backand Lambda Launcher is a tool we developed to ease the process of executing your AWS Lambda functions. Through use of the Lambda Launcher, you can easily execute any AWS Lambda function in your AWS account. This provides a number of benefits over traditional Lambda execution:

* You can provide users with a simple interface to your Lambda function library, allowing for execution of a function with a single click
* You can use non-AWS security systems to control user access to your AWS Lambda functions, allowing you to expose these functions without also needing to grant a user access to the AWS Console
* You can dynamically show and hide functions in the interface based upon user authentication and authorization, giving you full control over access to your Lambda functions
* You can integrate this functionality within a Backand application to give you ready-made devops, allowing you to perform reporting and maintenance tags from anywhere you have an internet connection

## Configuration

In order to work with your Lambda functions in Backand, you'll first need to connect Backand to your AWS account. Below, we walk through the process of creating a security policy to govern your integration of AWS Lambda with Backand, creating a user with the security access necessary to connect to your Lambda functions from Backand, and finally configuring Backand to communicate with your AWS Lambda functions.

These next sections focus on configuring the Lambda Launcher tool from a number of different locations. In all cases, the process will have three primary steps:

1. Create an IAM Policy to control access to your account's Lambda functions
2. Create an IAM User that you can use to connect Backand to your AWS account
3. Connect AWS to Backand using the IAM User's access keys

In all cases, step 3 - pasting the credentials into Backand - is the same, and as such we present the instructions for that step in a section after the AWS-specific configuration instructions. To create the AWS IAM Policy and User, choose the section most appropriate to your organizational practices from the options below.

### IAM Policy JSON
> The following JSON is to be used when creating the IAM security policy, to manage Backand's access to your AWS Lambda functions

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1495964947000",
            "Effect": "Allow",
            "Action": [
                "lambda:GetFunction",
                "lambda:InvokeAsync",
                "lambda:InvokeFunction",
                "lambda:ListFunctions"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

The Backand Lambda integration relies on the policy JSON provided to the right. This grants Backand the capability to fetch a list of functions, fetch an individual functions, and invoke a function both synchronously and asynchronously.

### From the AWS Console
If you're already logged into the AWS Console for your region, use the following steps to connect your AWS Lambda functions to Backand:

#### Create the IAM Security Policy
1. Open the IAM service panel
2. In the navigation menu on the left, select `Policies`
3. Click the `Create Policy` button
![image](aws_console/aws-IAM-Policy-add.png)
4. For "Step 1 : Create Policy," select `Create Your Own Policy`
5. Set the policy name to `BackandLambdaReadExecuteAccess`
6. Paste the [policy JSON](#iam-policy-json) into the area provided for the Policy Document
![image](aws_console/aws-IAM-Policy-add-3.png)
7. Once the JSON is ready, click `Create Policy`

#### Create the IAM User
1. On the navigation menu on the left, select `Users`
2. Click `Add User`
![image](aws_console/aws-IAM-users-add.png)
3. Enter `backand_lambda_user` as the user name
4. Check the `Programmatic access` option
![image](aws_console/aws-IAM-users-add-1.png)
5. Click on `Next: Permissions`
6. Click on `Attach existing policies directly`
7. Search for `BackandLambdaReadExecuteAccess`, and select it from the search results using the provided checkbox
8. Click `Next : Review`
9. click `Create User`
10. On the `Complete` page of the wizard, click the `Show` link in the table column `Secret access key`
11. Copy the "Access key ID" and "Secret access key" values to use with Backand.

### From the AWS Lambda Service panel
Using AWS Lambda, you can leverage a python script that we created which goes through the process of creating the IAM Policy and User on your behalf.

#### Lambda Python Script
```python
from __future__ import print_function

import json
import boto3

print('Loading function')

s3 = boto3.client('s3')


def lambda_handler(event, context):

    settings = {
        "name" :"backand_lambda_user",
        "policyName":"BackandLambdaReadExecuteAccess",
        "policy": ""
    }
    settings["policy"] = json.dumps({
          'Version': '2012-10-17',
          'Statement': [
            {
              'Sid': 'Stmt1494926726830',
              'Action': [
                'lambda:GetFunction',
                'lambda:InvokeAsync',
                'lambda:InvokeFunction',
                'lambda:ListFunctions'
              ],
              'Effect': 'Allow',
              'Resource': '*'
            }
          ]
        })
    client = boto3.client('iam')
    response = client.create_user(
        UserName=settings["name"]
    )
    print(response);
    accessReponse = client.create_access_key(
        UserName=settings["name"]
    )
    print(accessReponse)
    policy = client.create_policy(
        PolicyName= settings["policyName"],
        PolicyDocument=settings["policy"]
    )
    print(policy)
    user_policy = client.attach_user_policy(
        UserName=settings["name"],
        PolicyArn=policy["Policy"]["Arn"]
    )
    return {
        'AccessKey': accessReponse["AccessKey"]["AccessKeyId"],
        "SecretKey":accessReponse["AccessKey"]["SecretAccessKey"]

    }
```

This approach uses a python script to create the appropriate IAM resources. This script is provided in the right-hand panel, and should be used as the body of the Lambda action that will perform the required resource allocation in IAM.

#### Create the Lambda Function
1. go to Lambda service panel
2. Select the "Blank Function and click "Next"
3. name your function "createBackandLambdaUser"
4. Select the "Python 3.6" runtime
5. paste the [lambda python code](#lambda-python-script) into the provided area.
![image](aws_lambda/aws-lambda-config-1.png)
6. in the Role dropdown select "Create Custome Role"
7. in the "AWS Lambda requires access to your resources" window, select "Create new Role Policy"
8. click "View Policy Document" and click the Edit button on the right
![image](aws_lambda/aws-lambda-role.png)
9. approve the "Edit Policy" pop-up
![image](aws_lambda/aws-lambda-role-1.png)
10. Paste the [policy JSON](#iam-policy-json) into the area provided for the Policy Document and click "Allow"
![image](aws_lambda/aws-lambda-role-2.png)
11. click "Next"
12. click "Create  Function"
13. after the function was created , click "Test",
![image](aws_lambda/aws-lambda-config-2.png)
11. Copy the "Access key ID" and "Secret access key" values to use with Backand.

### From the AWS CLI
If you primarily use the AWS CLI to interact with AWS, you can easily connect Backand to your AWS account using the following steps:

#### Create the IAM Policy and User

First, ensure that all files involved are saved in an ASCII-encoded format. Before starting, you'll need to create a new file in the current directroy - `createBackandLambdaUser.json` - and populate it with the [policy JSON](#iam-policy-json) before starting. Then, run the following series of commands to create the IAM Policy and User in your account:

<aside class="notice">Be sure to replace &lt;&lt;ACCOUNT_NUMBER&gt;&gt; with your AWS Account Number.</aside>

1. aws iam create-policy --policy-name BackandLambdaReadExecuteAccess --policy-document file://createBackandLambdaUser.json ( and copy the policy arn for step 4)arn:aws:iam::<<ACCOUNT_NUMBER>>:policy/BackandLambdaReadExecuteAccess
2. aws iam create-user --user-name backand_lambda_user
3. aws iam attach-user-policy --user-name backand_lambda_user --policy-arn arn:aws:iam::<<ACCOUNT_NUMBER>>:policy/BackandLambdaReadExecuteAccess
4. aws iam create-access-key --user-name backand_lambda_user
5. Copy the "Access key ID" and "Secret access key" values to use with Backand.


### Connect the IAM User to Backand
If you are configuring your Lambda Launcher access for the first time, via the modal dialog, you can simply paste your Access Key ID and Secret Access Key into the provided fields. If you have an existing app, you can update those from the External Functions page:

1. Open the Backand App Dashboard
2. Select the `Functions & Integrations` tab, if it is not already selected
3. Select `External Functions`, under `Functions` in the left-hand navigation menu
4. Provide the Access Key, Secret Key, and Region in the provided fields
5. Press `Link`

At this point, your AWS Lambda functions should be listed in the Backand interface.
