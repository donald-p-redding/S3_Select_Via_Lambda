# Using S3 Select with AWS Lambda URL

## Context
This tutorial is intended to give a high level look at how AWS Lambda can interact with a user’s other AWS services.

The services we will be working with are AWS Lambda and AWS S3.

This tutorial will use Lambda Function URLs to make function calls. While a project will more than likely choose
to trigger lambdas in a different way, hopefully this tutorial will provide some hands on experience as to how lambdas
are built and configured so users can begin to think about how to do these tasks programmatically.

## Setup Steps
  1) Access your AWS console and create a new bucket with a unique name:
    Example: ```lambda-test-data``` (be sure to remember the bucket name).
    Keep all other settings to their default values. 
