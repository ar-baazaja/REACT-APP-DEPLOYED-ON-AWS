# 🦄 AWS Project - Wild Rydes Full-Stack Web Application (End-to-End Tutorial)

This repository contains all the code and setup I used to build the **Wild Rydes** serverless web application, following the original [Amazon Workshop](https://aws.amazon.com/serverless-workshops) and [this video tutorial](https://youtu.be/K6v6t5z6AsU).

## 🚀 What This Project Does

The application lets users:
- Sign up and log in via **Cognito**.
- Click on a map to request a unicorn ride (via **ArcGIS**).
- Trigger a **Lambda function** to assign a unicorn.
- Store the ride info in **DynamoDB**.
- Use **API Gateway** to expose the backend.
- Deploy the app using **Amplify** and manage CI/CD via GitHub.

All components are integrated into a seamless serverless app using **7 AWS Services**:
1. Cognito
2. IAM
3. API Gateway
4. Lambda
5. DynamoDB
6. Amplify
7. GitHub (CI/CD)

---

## 📸 Screenshots

### Cognito User Pool Setup
![Cognito Screenshot](./Capture.png)

### Amplify Deployment
![Amplify Screenshot](./Capture2.png)

### API Gateway Configuration
![API Gateway Screenshot](./Capture3.png)

### Lambda Function
![Lambda Function Screenshot](./Capture4.png)

---

## 🧪 Technologies Used

| Component      | Service Used            |
|----------------|-------------------------|
| Authentication | AWS Cognito             |
| Backend Logic  | AWS Lambda              |
| API Layer      | Amazon API Gateway      |
| Data Storage   | Amazon DynamoDB         |
| Hosting & CI/CD| AWS Amplify + GitHub    |
| Map UI         | ArcGIS JavaScript SDK   |

---

## 🛠️ How I Built It

### Step-by-Step Process

1. **Set up Cognito** for user sign-up/sign-in.
2. **Created a Lambda function** to assign a unicorn and log the ride.
3. **Used API Gateway** to trigger the Lambda function.
4. **Configured DynamoDB** to store ride info.
5. **Set up Amplify** to deploy the front-end and connect to GitHub.
6. **Built the front-end app** using HTML/CSS/JavaScript with ArcGIS.
7. **Integrated everything** into a secure, working app.

---

## 📂 Lambda Function Code

Here's the main Lambda handler used to assign unicorns:

```js
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;
    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
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
        };
    } catch (err) {
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64').replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({ Error: errorMessage, Reference: awsRequestId }),
        headers: { 'Access-Control-Allow-Origin': '*' },
    };
}
