# **Introduction**

Company Canary Deployment is a project to simulate the benefits of using deployment strategies like canary in a serverless (AWS) environment, using SAM (serverless application model) to create, build, deploy and publish all infraestructure.

# **Pre-requisites tools version (or newer)**
- SAM CLI 1.53.0
- AWS CLI 2.7.19

# **Solution Architecture**

It was createad a very simple structure simulating a company manager form, in this case we have a CRUD for create, read, update and delete company data. To save the data it has a AWS Dynamodb database, the rules and execution is in AWS Lambda functions, and for last, the interface for these services, we have a API Gateway. Until here nothing complicated and very simple to understaind.

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/Architecture/Architecture.png)

But, the main point of this project is testing a deployment stragy, and for that, using AWS SAM to manage the application and for every Lambda function we have the following architecture:
For every deploy in a AWS Lambda function, the SAM uses AWS Code Deploy to deploy the application, and before it releases, it invokes an other Lambda function (Lambda PreTraffic) to execute the unit tests, and if it fails, automatically stop the deployment and rollback to previous version.

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/Architecture/Architecture%20Deploy.png)

If the PreTraffic goes sucess, then it starts to deploy the application in canary deployment.
*"A canary deployment, or canary release, is a deployment pattern that allows you to roll out new code/features to a subset of users as an initial test."*

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/codedeploy1.png)

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/codedeploy2.png)

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/codedeploy3.png)

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/codedeploy4.png)

In this case, we rollout 10% for every one minute between the previous (original) and newer (replacement) version, it means the users of the services, will see the product besead in the proportion of the percentage rolling up in the deployment, as it is showing in the images above.
In the SAM template, it is possible to create alarms for any metric conected with the code deploy, it means, if this alarm goes in "alarm" stage, the deploy will stop and rollback automatically. In this case, the alarm defined is Lambda erros, if any errors ocurs, the alarm will be upped.

Finally in the deployment architecture we have the PostTraffic Lambda function, to execute the role integration/functionally tests of the product, it means, even if the unit test pass, and all the deployment pass, if the integration/functionally test (PostTraffic) fail, the deployment will stop and rollback automatically too.
To execute these test, it is using AWS Step Function (Express workflow synchronously) to call all the routes of API Gateway and test all functionally of the product. 
The AWS Step Functioions description is showed in the image below:

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/stepfunctions_graph_studio.png)

As mentioned many times, if any step fail, the deployment will rollback, the image below illustrate this scenario if it happens:

![alt text](https://raw.githubusercontent.com/markoshlima/company-canary-deployment/main/docs/codedeploy-fail.png)

# Result

Working with the deployment strategy, we are improving the reliability of the product, which means ease in deploying and launching new capabilities, in addition to reducing service unavailability due to a deploy or human error, this directly affects the success of any product.
Working with SAM brought a significant ease in the evolution and ease of creating the product, making the team focus more on the business than on configurations.

# Addition Information & Setup

To setup the environment:
- The `IaaC` folder contains the scripts to create the database and S3 bucket (to save the sam files)
- the `open-api` folder there is the Open API specification to create the gateway
- in the `statemachine` folder contains the workflow to create the AWS step functions process.
- Inside each microservices run the SAM CLI to create, build, deploy and publish each function.