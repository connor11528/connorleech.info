---
layout: post
title: Tutorial for building a Web Application with Amazon S3, Lambda, DynamoDB and API Gateway
date: "2017-08-28"
categories: ["Cloud"]
toc: false
description: "In this tutorial we build out a ride sharing application utilizing jQuery, Node.js and AWS Serverless architecture for fun and profit."
author: Connor Leech
published: true
---

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/wildrydes-complete-architecture.png)

I recently attended Serverless Day at the AWS Loft in downtown San Francisco. During the workshop section we built a serverless web application for requesting Unicorns to come pick us up. The AWS team provided excellent documentation [on Github](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication) and [Rahul Sareen](https://www.linkedin.com/in/rahul-sareen-1b2bb86/) gave a one of the best presentations I have heard at a tech event overviewing Serverless application architecture. (Slides for that presentation are [available here](https://www.slideshare.net/AmazonWebServices/getting-started-with-aws-lambda-and-serverless-computing-79032206)).

In the workshop portion we created and deployed a website that utilized S3 for hosting, DynamoDB for a database, API Gateway for RESTful endpoints and Lambda functions as our backend server processing.

This tutorial covers my notes from building out the application and using some of these services for the first time on Serverless Day 2017. More detailed notes for following along are available [on the github](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication) and the Wild Rydes demo application is live at http://www.wildrydes.com/.

## Step 0: About WildRydes

The application we are going to create in this tutorial is called Wild Rydes. The application is a fictional service for ordering unicorns to come pick us up. Users can login to the application and request unicorns from their current location. The application then dispatches a unicorn to pick up the user.

![](http://www.wildrydes.com/images/wr-home-top.jpg)

Without further ado, let's get started.

## Step 1: Identity Access Management

As with most AWS tutorials, the first step is to create an IAM user that will create and provision our AWS resources. I have a user set up that has AdminAccess. It is considered best practice to login using such an user rather than logging into and managing your AWS resources using your root account credentials. If you have no idea what I'm talking about I suggest checking out the [A Cloud Guru course](https://acloud.guru/course/aws-certified-developer-associate) for passing the AWS Certified Developer - Associate exam. Chapter 3 provides easy to follow video instructions on setting up users for your AWS account.

If you are not so inclined, the AWS team also provides [detailed instructions](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/3_ServerlessBackend) for creating an IAM user with the specific permissions (`AWSLambdaBasicExecutionRole`) to write to DynamoDB and CloudWatch. If you associate your Lambda function with a user that has admin access your Lambda function will be able to access any service.

You also want to make sure that when you install the [AWS CLI](https://github.com/aws/aws-cli) it is associated with the user you created. When creating a new IAM user you get one chance to download the key-value pair for that user. In the command line type `aws configure` and you can set your public and secret API keys for the CLI.

Managing user access is important for account security and provisioning access to our AWS resources. We ran into some errors getting things set up and all of the errors were related to IAM so make sure you have permissions to do what you are trying to do! (*pro tip*: `aws configure` helps)

## Step 2: Static Website on Simple Storage Service (S3)

In this section of the tutorial we are going to create an S3 bucket to host the static portion of our Wild Rydes application. Static Website means HTML, CSS, Javascript and Image files. S3 provides **object storage** meaning we cannot run an operating system on it but we can host a website.

The first step is to create an S3 bucket and enable the static web hosting option for that bucket. The AWS team provides details instructions on how to do this [here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/1_StaticWebHosting).

When static website hosting is enabled for an S3 bucket, the contents of the **index.html** file within that bucket will be publicly accessible to the internet following this URL structure: `http://BUCKET_NAME.s3-website-REGION.amazonaws.com/` where BUCKET_NAME is the globally unique name you gave your bucket and REGION is the region you created the bucket in (such as `us-east-1` for Virginia or `us-west-2` for Oregon).

Since this tutorial focuses on AWS infrastructure instead of static website coding, we copy the files for Wild Rydes from the AWS team. This code is open source and [available here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/1_StaticWebHosting/website)

The command to copy the contents of their bucket into our bucket is as follows:

```
aws s3 sync s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION
```

After running this command all of our static files should appear in our S3 bucket when we refresh the page showing our bucket contents. If you are having issues syncing the files across buckets using the command line make sure you are logged in as the same IAM user that created the bucket or that the keys/permissions line up.

Of the new contents of our bucket, the main file to take note of is **js/config.js**. We'll be editing this file with values from [Cognito](https://aws.amazon.com/cognito/) and [API Gateway](https://aws.amazon.com/api-gateway/). 

Finally, we want to make sure that our bucket is publicly accessible to the internet. For this we add a bucket policy as outlined below:

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/update-bucket-policy.png)

JSON schema for our S3 bucket policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
        }
    ]
}
```

My bucket is called **wildrydes-082317** and created within **us-west-2** (Oregon) so my static website files are publicly accessible here: http://wildrydes-082317.s3-website-us-west-2.amazonaws.com/

## Step 3: User Management with Cognito

In the [next step](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/2_UserManagement) we will configure a Cognito user pool to manage users. This hooks up the functionality for users to create 
 accounts, verify their email addresses and sign in to the Wild Rydes site.

Following the above instructions, the first step is to create a Cognito user pool using the AWS console. Cognito user pools provide out of the box functionality for federated identity providers (such as Google and Facebook login), password recovery and user authorization security in the cloud. You can learn more about user pools [here](http://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html). 

When we create our Cognito user pool and create an app client. App clients have permission to call unauthenticated APIs (such as register, login and forgot passowrd). Take note of your **Pool Id** and the **App client id** (featured below) as we will insert these values into **js/config.js**

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/pool-id.png)

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/client-id.png)

Head into your S3 bucket, download and modify **js/config.js** with your appropriate values from Cognito. Reupload the file back to your S3 bucket. We will have to do this one more time to populate the `invokeUrl` with a value from API gateway. Populating the `cognito` javascript object in that file connects our static web application to Amazon's cloud authentication services. For a detailed jQuery implementation of user management on the client side, view the files [here](https://github.com/awslabs/aws-serverless-workshops/blob/master/WebApplication/1_StaticWebHosting/website/js/cognito-auth.js).

Once we have updated our Cognito object within the config file, head over to the register page at `YOUR_S3_URL/register.html`. In my case the full url is: `http://wildrydes-082317.s3-website-us-west-2.amazonaws.com/register.html`.

Sign up and create an account. Use your real email address! Cognito sends a test email with a link to verify your account. When you check your email after creating your account you will have a verification code, such as: `211658`.

Go to `YOUR_S3_URL/verify.html` and enter your email address and confirmation code.

Go to signin page and signin with your new account: `/signin.html`

This flow could definitely be optimized. There is no client side routing implemented and we still have .html appended to all of our routes. Nevertheless, you can update this code with The Javascript Framework Of Your Choice. The backend process for registering users to Cognito will stay the same as we are using the Cognito client side JS SDK. The email verification is an option enabled by default that can easily be switched off. 

You can customize the verification message by navigating to your Cognito User Pool by clicking **Message Customizations** on the left navigation panel. 

It is worth noting here that we could use other authentication services such as [Auth0](https://auth0.com/) (they have an awesome developer blog). This is an Amazon provided tutorial though so we are using all AWS functionality.

When we successfully create a user, verify and sign in we will get to this screen:

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/successful-login.png)

## Step 4: Set up Serverless Backend

In this step we'll implement a Lambda function that will be invoked each time a signed in user requests a unicorn. Lambda functions are the core functionality qualifying apps as Serverless. Lambda functions are a managed service provided by Amazon. We provide the code for the Lambda function and only pay for the time it takes that function to execute. We do not have to deal with provisioning EC2 instances or Elastic Load Balancing (typical operations functions for cloud applications). The primary advantage of this approach is that is is far cheaper than dedicated cloud hosting. It can also allow us to focus more on writing code and less on operations. Serverless and Lambda functions are a new Amazon service and new paradigm for web applications so there will be a learning curve but have the potential to save us massive time and money down the road.

The full steps for setting up the serverless backend are [available here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/3_ServerlessBackend).

Before we even get to setting up Lambda functions and a serverless application we are going to create a DynamoDB database. [DynamoDB](https://aws.amazon.com/dynamodb/) is Amazon's managed NoSQL database. We are going to use DynamoDB to store information about the ride request when a user requests a Unicorn.

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/ddb-create-table.png)

When we create the database note the ARN. It will look something like this: 

```
Amazon Resource Name (ARN)	arn:aws:dynamodb:us-west-2:XXXXXXXXXXXX:table/Rides
```

Now that the database is created we're going to an IAM role for the Lambda function. Every Lambda function must have an IAM role associated with it. The IAM role defines what AWS services the Lambda function is permitted to interact with. In this case we are going to go with the `AWSLambdaBasicExecutionRole`. This basic role covers the functionality we need for the Wild Rydes application -- *writing logs to Amazon CloudWatch and writing items to a DynamoDB table*.

Detailed steps are [available here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/3_ServerlessBackend) for creating the IAM role.

Now that we have the DynamoDB database created and a role ready to associate with our Lambda function we can create the function itself!

Create a Lambda function called `RequestUnicorn`. The Amazon Web Services team provided the Node.js script for the Lambda function [here](https://github.com/awslabs/aws-serverless-workshops/blob/master/WebApplication/3_ServerlessBackend/requestUnicorn.js). The full code for our Lambda function is below:

```
const randomBytes = require('crypto').randomBytes;

const AWS = require('aws-sdk');

const ddb = new AWS.DynamoDB.DocumentClient();

const fleet = [
    {
        Name: 'Bucephalus',
        Color: 'Golden',
        Gender: 'Male',
    },
    {
        Name: 'Shadowfax',
        Color: 'White',
        Gender: 'Male',
    },
    {
        Name: 'Rocinante',
        Color: 'Yellow',
        Gender: 'Female',
    },
];

exports.handler = (event, context, callback) => {
    if (!event.requestContext.authorizer) {
      errorResponse('Authorization not configured', context.awsRequestId, callback);
      return;
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    // Because we're using a Cognito User Pools authorizer, all of the claims
    // included in the authentication token are provided in the request context.
    // This includes the username as well as other attributes.
    const username = event.requestContext.authorizer.claims['cognito:username'];

    // The body field of the event in a proxy integration is a raw string.
    // In order to extract meaningful values, we need to first parse this string
    // into an object. A more robust implementation might inspect the Content-Type
    // header first and use a different parsing strategy based on that value.
    const requestBody = JSON.parse(event.body);

    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    recordRide(rideId, username, unicorn).then(() => {
        // You can use the callback function to provide a return value from your Node.js
        // Lambda functions. The first parameter is used for failed invocations. The
        // second parameter specifies the result data of the invocation.

        // Because this Lambda function is called by an API Gateway proxy integration
        // the result object must use the following structure.
        callback(null, {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        });
    }).catch((err) => {
        console.error(err);

        // If there is an error during processing, catch it and return
        // from the Lambda function successfully. Specify a 500 HTTP status
        // code and provide an error message in the body. This will provide a
        // more meaningful error response to the end client.
        errorResponse(err.message, context.awsRequestId, callback)
    });
};

// This is where you would implement logic to find the optimal unicorn for
// this ride (possibly invoking another Lambda function as a microservice.)
// For simplicity, we'll just pick a unicorn at random.
function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

function recordRide(rideId, username, unicorn) {
    return ddb.put({
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    }).promise();
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}
```

Currently we can write Lambda functions in Node.js, Python, Java or C#. The above code is a Node.js function that checks the user is authorized, writes to DynamoDB within the `recordRide` function and sends a random Unicorn back to the user. After reviewing the code, paste in the Lambda function and create it, leaving the default `index.handler`.

We can also configure a test event to make sure our Lambda function is envoked properly. If you would like to test your Lambda function, paste in the sample [event code](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/3_ServerlessBackend) and verify that the execute succeeds.

## Step 5: Setup API Gateway

We have set everything up for our Lambda function and static website. Now we need to set up API Gateway so that our static website can trigger the Lambda function. Amazon's [API Gateway](https://aws.amazon.com/api-gateway/) allows us to create RESTful APIs that expose HTTP endpoints. These endpoints can be invoked from the browser.

The final step is to [create an API Gateway](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/4_RESTfulAPIs) that will be our REST API. We could use tools like [Swagger](https://swagger.io/) or [stoplight.io](http://stoplight.io/) at this point. Since we are only creating one HTTP endpoint we will create it manually.

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/create-api.png)

After creating the API Gateway, we hook up Cognito to our endpoints. Doing this allows API Gateway to use and test the JWT tokens returned by Cognito. If you are not familiar with JWT, you can check out a sample applications [here](https://github.com/connor11528/vuejs-auth-frontend) and [here](http://connorleech.info/blog/use-express-angular-and-jwt-to-make-a-secure-app/) utilizing client side Javascript.

In order to hook up Cognito to API Gateway and protect our endpoints create a Cognito User pool authorizer:

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/create-user-pool-authorizer.png)

Select Authorizers. Create -> Cognito user pool.

Now that that is configured we create a new **resource method** for the `POST /ride` endpoint.

![](https://github.com/awslabs/aws-serverless-workshops/raw/master/WebApplication/images/api-integration-setup.png)

More detailed instructions are available [here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication/4_RESTfulAPIs) but the gist is that we select the option for Proxy Integration and add the WildRydesLambda function tat we created in the last step. Select method request card and under authorization select our Cognito user pool.

We also have to enable CORS for our endpoint. In the API Gateway console, under **Actions** and replace default values and select **Enable CORS**. Everything can be left as the defaults.

Deploy the API Gateway by selecting **Actions -> Deploy**. This generates an **Invoke URL** that we must include in **js/cofig.js**. In my case the value is `https://tfyxh265h2.execute-api.us-west-2.amazonaws.com/prod`. This endpoint is what our website requests via AJAX that invokes the Lambda function.

Everything should work now. The demo application is [available here](http://wildrydes-082317.s3-website-us-west-2.amazonaws.com/). If you have any questions about Node.js or serverless I'm available [on twitter](https://twitter.com/Connor11528) and the full source code from the AWS team is [here](https://github.com/awslabs/aws-serverless-workshops/tree/master/WebApplication)

Thanks for reading! If you enjoyed please share/upvote so that more people can hop on the serverless bandwagon and drink the Kool Aid.

![Serverless Kool Aid](https://media3.giphy.com/media/VxoZWcyfgbzhe/giphy.gif)

