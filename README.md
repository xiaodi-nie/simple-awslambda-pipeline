## A simple data engineering pipeline with AWS Lambda and SQS

This is a Duke ECE590-24 course project. In this project I reproduced a simple serverless data engineering pipeline proposed by our professor. It looks something like this:

![pipeline](img/serverless-pipeline.png)

First a producer lambda function will be invoked every minute, and it would access a DynamoDB table, grab the entries(which are a bunch of company names), and send them as messages into a SQS queue. Every time there're new messages coming into that SQS queue, a consumer lambda function would be invoked. The consumer would get the first line of the wikipedia result of each of the messages, and do a sentiment analysis with it via AWS Comprehend. The results would then be put into a S3 bucket in csv files.

### General Steps

**Create Producer Lambda Function with Cloud9**

- Create a basic Cloud9 environment and connect
- On the right panel find `AWS Resources - Lambda - Create a new Lambda function`
- In the wizard choose `python3.6` as the runtime, leave the trigger as `none`, and remember to give it a role with `AdministratorAccess`
- Paste in the producer code
- Install 3rd party packages

    The good thing about lambda python environment is that it comes with a virtual environment. Just activate it and install the packages to the right folder.
    ```
    cd ~/environment/consumerLambda/consumerLambda
    source ../venv/bin/activate
    sudo /usr/bin/python3 -m pip install --upgrade pip #update pip3
    pip3 install python-json-logger --target ../
    ```
- Create the corresponding DynamoDB table and primary key
    
    In the code the table name is `company-names` and the primary key is `guid`. Remember to change these two in the source code accordingly.
- Create the corresponding SQS queue. Just leave everything default and remember to change the queue name in the source code
- Run the function locally(in Cloud9) and test if everything is doing ok
- Right click on your function name from the right panel and `Deploy` to the cloud
- Go to the Lambda service on the console
- Find the function we just deployed and add a trigger:
    
    Choose `CloudWatch Event`, create a new rule and use `rate(1 minute)` as the `Schedule expression`, enable trigger


**Create consumer Lambda function with Cloud9**

- Create a S3 bucket, leave everything default, and remember to change the bucket path in the consumer source code
- Repeat basically the same thing, change the queue name in the source code
- Of course install different 3rd party packages: `python-json-logger`, `wikipedia`, and `pandas`
- And a different trigger:

    This time choose `SQS`, select your queue name, and leave the batch size default(or not)
    
You can check the S3 bucket for results and CloudWatch for logs. The logging in the code is at the `info` level, you can also change it to `debug` for more detailed information logging.
