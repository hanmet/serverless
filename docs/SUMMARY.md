# Summary of course:

### Function as a Service

#### Serverless Components
- FaaS : Function as a service: write code in individual functions and deploy them to a platform to be executed
- Datastores: Storage of data
- Messaging: Send messages from one application to another
- Services: Services that provide functionalities where we don't need to manage servers; i.e. authentication, ML, video processing

#### Function as a Service
- Split application into small functions
- Event driven
- Pay per invocation
- Rest is handled by a cloud provider

#### Lambda function vs AWS Lambda
Keep in mind that AWS Lambda is a computing service that runs code in response to events from Amazon Web Services, while a Lambda function is a single function connected to an event source running in AWS Lambda.

#### AWS Lambda limitations

- At most 3GB of memory per execution
- Functions can run no more that 15 minutes
- Can only write files to /tmp folder
- Limited number of concurrent executions
- Event size up to 6 MB
- Keep in mind that those limits are subject to change. You can find most recent AWS Lambda limits [here](https://docs.aws.amazon.com/lambda/latest/dg/limits.html).


### AWS Lambda

Sample code of a function with no parameters.

```javascript
exports.handler = async (event) => {
        console.log('Event: ', event);

        return {
               result: 'Hello World!'
       }
}
```

Sample code of a function with parameters.

```javascript
exports.handler = async (event) => {

        console.log('Event: ', event);
        return {
               result: `Hello ${event.name} !`
       }
}
```


#### How functions are executed
When we send a request to execute a Lambda function, AWSLambda creates an environment to run that function. It starts a container for the specific environment and loads the function code into the environment. Then it sends an event to our function. The same process is repeated for all the other requests coming in.


### Invocation types
- Request / Response
- Asynchronous invocation
- Using AWS CLI


Sample for AWS CLI returning a response:
```shell script
aws lambda invoke --function-name hello-world \
 --invocation-type RequestResponse \
 --cli-binary-format raw-in-base64-out \
 --log-type Tail --payload '{"name": "AWS Lambda"}' \
 result.txt
``` 

Only returning a status code and executing the function asynchronously:
```shell script
aws lambda invoke --function-name hello-world \
 --invocation-type Event \
 --cli-binary-format raw-in-base64-out \
 --log-type Tail --payload '{"name": "AWS Lambda"}' \
 result.txt
``` 

#### Error handling

Errors are handled differently, depending on how we execute our function.

When we use a Request/response method: If there's an error in the function, then it will return immediately to the caller, which can process the error from the Lambda function.

When we use an Async method: Instead of returning an error to the user, AWSLambda will return HTTP 202 code to the user and it will store a request into an internal queue. Additionally, it will try to call the Lambda function up to 3 times. If all of those times result into an error, then it will store the event into a "dead-letter queue", which stores all the events that the Lambda function failed to process.

### Deploy packaged lambda function

Example repository: [chaos monkey](https://github.com/udacity/cloud-developer/tree/master/course-04/exercises/c4-demos-master/04-chaos-monkey)

In the 04-chaos-monkey directory install the dependencies:
`npm install`

and then zip the folder so that it can be uploaded.
`zip -r chaos-monkey.zip .`

AWS console -> Lambda -> Create new function -> Upload a .zip file

### IAM
By default lambda function can only write to cloudwatch logs. If they need access to other AWS resources, we need to change the IAM role of this lambda function.

### Additional Parameters

| Field | Description |
| ------|----------- |
|functionName | Name of function |
|functionVersion | Specific version of a function |
|memoryLimitInMB | Maximum amount of memory available |
|logGroupName | log group for the function |
|logStreamName |log stream for the function |
|getRemainingTimeInMillis() | Get remaining time in ms |


## REST API

Who sends an event to a lambda function, if we have no ec2 instances in our infrastructure?
- API Gateway

What is API Gateway:
- Entry point for API users
- Pass requests to other services
- Process incoming requests

### Amazon API Gateway:
#### Amazon Gateway API Types:
- Rest API
    - Regular HTTP API with resources and methods
    - Request / Response nature
- WebSocket API
    - For real time communication between server and client
    - Requires persistent connection
    
#### API Gateway targets

Possible targets for an HTTP request processed by API Gateway:

- Lambda Function - call a Lambda function
- HTTP Endpoint - call a public HTTP endpoint
- AWS Service - send a request to an AWS service
- Mock - return a response without calling a backend
- VPC Link - access resource in an Amazon Virtual Private Cloud (VPC)



#### Endpoint types in API Gateways

- Edge optimized (with Cloudfront in front)
- Regional (no Cloudfront in front)
- Private (for requests inside of a network in AWS)

#### Lambda integration modes
     
- Proxy - passes all request information to a Lambda function. Easier to use.
- Non-proxy - allows to transform incoming request using Velocity Template Language

### DynamoDB

#### DynamoDB Capacity Modes

DynamoDB has two capacity modes:

- Provisioned capacity: we need to define the maximum amount of read/write requests DynamoDB can handle. The higher the limit we set, the more we have to pay per month. Requests are throttled if we go above the specified limit.
- On-Demand: DynamoDB will handle as many requests as we send, and we pay per-request. Can be more expensive comparing to Provisioned capacity, but is better for applications with unpredictable traffic patterns


#### Sample
Sample code to read data from DynamoDB

```javascript
const docClient = new AWS.DynamoDB.DocumentClient();
 // ...
 const result = await docClient.scan({ // Call parameters
   TableName: "Users",
   Limit: 20
 }).promise()
```

### Serverless Framework

#### Setup Serverless Framework

1. Install serverless:
`npm install -g serverless`

2. Set up a new user in IAM named "serverless" and save the access key and secret key.

3. Configure serverless to use the AWS credentials you just set up:
```sls config credentials --provider aws --key YOUR_ACCESS_KEY --secret YOUR_SECRET_KEY --profile serverless```

#### Setup infrastructure with Serverless Framwork
To get a list of the available serverless templates run:

`sls create --template`

To create a serverless boilerplate project:

`sls create --template aws-nodejs-typescript --path 10-udagram-app`

To deploy the application:

1. Install the dependencies: `npm install`
2. Deploy the application `sls deploy -v`

