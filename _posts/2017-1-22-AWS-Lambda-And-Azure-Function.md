---
layout: post
title: Developing AWS Lambda and Azure function in C#. 
---

# AWS Lambda
[AWS Lambda](https://aws.amazon.com/lambda) is an serverless computing platform provided 
by Amazon that lets one run code without provisioning or managing servers. It was introduced in 2014.

The code you run on AWS Lambda is called a “Lambda function.” Initially, AWS Lambda supports code written 
in Node.js (JavaScript), Python, Java (Java 8 compatible). In 2016, Amazon added support C# (using the 
.NET Core runtime running on Linux). Lambda can be triggered by changes in data, shifts in system state, 
or actions by users. Lambda can be directly triggered by AWS services such as S3, DynamoDB, 
Kinesis, SNS, and CloudWatch. To expose Amazon Lambda to the internet, one uses 
[Amazon API Gateway] (https://aws.amazon.com/api-gateway/).

To develop AWS Lambda in C#, one first downloads and installs 
[AWS Toolkit for Visual Studio] (https://aws.amazon.com/visualstudio/). 
The toolkit has two project templates for recreating Lambda functions: 

* The AWS Lambda Project template creates a simple project with a single C# Lambda function. 
* The AWS Serverless Application template creates a small AWS serverless application, following the AWS Serverless Application Model (AWS SAM). 
One can develop a complete serverless application composed of multiple Lambda functions exposed through an API Gateway REST endpoint.

