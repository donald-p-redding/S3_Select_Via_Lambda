# Using S3 Select with AWS Lambda URL

## Context
This tutorial is intended to give a high level look at how AWS Lambda can interact with a user’s other AWS services.

The services we will be working with are:
* AWS Lambda
* AWS S3.

This tutorial will use ```Lambda Function URLs``` to make function calls. While a project will more than likely choose
to trigger lambdas in a different way, hopefully this tutorial will provide some hands on experience as to how lambdas
are built and configured so users can begin to think about how to do these tasks programmatically.

## Assumptions
This tutorial will assume that users are working with a node version ```14.x```
and this will influence how we build our lambda.

If you do not currently have a node version matching ```14.x``` it is recommended to reference [Node Version Manager](https://github.com/nvm-sh/nvm) before moving forward.

## Basic Setup Steps
  1. Clone this repo to your local machine.
  2. Access your AWS console and create a new bucket with a unique name.
  * Example: ```lastname-lambda-test-data``` (be sure to remember the bucket name).
  * Keep all other settings to their default values. 

  3. Access the dashboard for your newly created bucket:
  * Upload the log ```demo-access.log``` within the  ```/data``` directory into your bucket.

  4. Access your IAM console within the AWS web dashboard:
  * Click ```Roles``` and then click ```Create role```
  * Ensure Trusted entity type is ```AWS Service``` and ```Common use case``` is set to Lambda.
  * Move to the next step in the console.
  * Add the following permissions to the Role:
    * ```AmazonS3ReadOnlyAccess``` -- allows our future lambda access to our buckets
    * ```AWSLambdaBasicExecutionRole``` -- will allow lambda to write logs to CoudWatch for debugging.
  * Move to the next step in the console.
  * name the role ```lambda-s3-access```
  * Confirm changes and create the role.

  5. Access the AWS Lambda console.
  * Click ```Create function```
  * Name your lambda function ```queryS3```
  * Set Runtime to ```Node.js 14.x```
  * Keep Architecture set to ```x86_64```
  * Under the Permissions heading, select ```Change default execution role```
    * Click ```Use an existing role```
    * Select the role we created in step 3. ```lambda-s3-access```
  * Click ```Create Function```


## Sanity Check
At this point you will have created:
  * An S3 bucket filled with some demo data.
  * A role that will allow our lambda the following permissions:
    * ```AmazonS3ReadOnlyAccess```
    * ```AWSLambdaBasicExecutionRole```
  * A lambda with a ```Node.js 14.x``` runtime environment.

In order to easily interact with our S3 bucket, we will be using the node pacakge ```@aws-sdk/client-s3```

This is where things get a little tricky. Out of the box, lambdas will not have access to node packages at runtime. In order to allow our lambda to use ```@aws-sdk/client-s3``` we will have to introduce it into the lambda's runtime environment.

## Advanced Setup
Thankfully AWS now allows developers to create ```Layers``` for their lambdas. We can create a Layer within our AWS console and this will act as a runtime enviornment or context where a lambda can operate. Giving it access to the desired node packages.

The benefit of Layers is that you can load a layer full of node packages and allow any number of lambdas to operate in this environment.

This repo has done the directory layout for you, but it is important to note:

***Your layer must be contained in a directory named ```nodejs```.***

## Steps to Create a Layer
  1. Navigate to the ```/nodejs``` directory
  * You'll find a ```package.json``` file already in the directory
  * Within ```/nodejs``` run ```npm install```
  2. Navigate back to the root directory of this repo.
  3. Zip the entire ```/nodejs``` directory and its contents.
  * Problems have been reported with simply right clicking and zipping
  * On Mac you can run
```bash
zip -r nodejs.zip nodejs
```
  4. A zip file will be created that contains all the packages our ```Layer``` will need.

## Steps to Apply a Layer to Lambda
  1. Navigate back to the AWS lambda dashboard and select ```Layers``` under Additional resources.
  2. Click ```Create Layer```
  * Name your layer ```query_s3```
  3. Upload the zip file that's now in the root directory of this local repo.
  4. Select ```x86_64``` as your Compatible architectures (optional) -- pending confirmation
  5. Select ```Node.js 14.x``` under Compatible runtimes (optional) -- pending confirmation
  6. Click create.
  7. Navigate to your lambda function.
  * You'll see a ```Layers``` tab under the lambda name in the Function overview section of the dashboard.
  * Click ```Layers``` and then ```Add a Layer```
  * Select Custom Layers and select your recently created layer, then click ```add```

## Adding Code to our Lambda
  1. Take the contents of ```/src/lambdaCode.js``` and copy into the ```Code``` section of your lambda in the Lambda dashboard.
  2. Save the current state of the source code.
  3. Click ```deploy```

## Calling our lambda
The last step is to call our lambda and pass it some parameters so that it can query our S3 bucket.

For simplicity we will create a ```Function URL``` to be able to invoke our lambda via a HTTP client like Postman.

1. Within the same dashboard that we pasted and deployed our lambda source code, click on the ```Configuration``` heading.
2. Navigate to ```Function URL``` and click Create Function URL
3. For demo purposes, select the ```NONE``` Auth type.
4. Also enable (CORS) for your lambda.
5. Click ```Save```.
6. Copy the function URL that was generated.
7. Open Postman or a similar HTTP client and create a new GET request.
8. Paste the lambda function URL as the request URL
9. Include a body with the request that conforms to the following structure.

```json
{
    "Bucket": "bucket-name-created-in-BasicSetup#1",
    "Key": "demo-access.log"
}
```

If all goes well you should see a JSON payload returned containing the first 5 log files within ```demo-access.log```.

If all goes wrong, you will recieve a payload containing the error object that was generated during the lambda's runtime.

## Wrap up
Hopefully this walkthrough provided some insight as to how to build lambda functions to interact with our other AWS services and start to spark questions as to how these steps might be handled progrmatically in a SDK or via the AWS CLI.

    


