# Step 1- Create your Lambda function

In the first step you will create an AWS Lambda function that will run in the cloud and filter the messages that are coming from your DeepLens device for ones with high enough (>0.5) probability for a 'person'. You can play with the probability as part of this session. During this process you will also create a rule in the AWS IoT rule engine to get messages from the Lambda function that you deployed to the device using AWS Greengrass.

1. Login to Lambda console (https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions). Make sure you are on US East (N. Virginia)

2. Click on Create function

3. In the blueprints search bar, search for "iot-button-email"

4. Choose the template

# Step 2- Name your lambda function and attach policies

1. Give the lambda function a name: iot-trigger-uniqueID

2. Keep “Create a new Role from template(s)” value in the Role field

3. Give your new role the same name as your lambda function name. 

4. In the policy templates, search for “SNS Publish policy”. Add the policy

# Step 3- Create IoT rule

1. In the “aws-iot” section switch to use “Custom IoT Rule”

2. Choose “Create a new rule”

3. Give it a name: person-trigger-uniqueID and a description

4. In the Rule query statement put the SELECT query: Select person from '$aws/things/deeplens_xxxxx-xxxxx/infer'

Replace $aws/things/deeplens_xxxxx-xxxxx/infer with the IoT topic for your AWS DeepLens. You can find it by navigating to Devices in your AWS DeepLens and choosing your device name. Scroll down to the bottom of the device detail page to find your IoT topic. The below query can catch messages from the AWS DeepLens device in the following JSON format: { “person” : 0.5438 }

# Step 4- Enable Trigger and change environment variables

1. Enable trigger in the checkbox

2. Change the environment parameter from "email" to "phone_number", and put your phone number as the Value. Please note that the phone number format should include the international country code (for example, for US +15555555555). 

3. Click the Create function button

# Step 5- Change configuration

1. Switch to the “Configuration” tab of the Lambda function that you just created. You can find configuration tab on the left (configuration, triggers and monitoring)

2. In the lambda function code, remove all the existing code and copy paste the code below

```python
'use strict';

/**
 * This is a sample Lambda function that sends an SMS Notification When your
 * Deep Lens device detect a Hot Dog
 * 
 * Follow these steps to complete the configuration of your function:
 *
 * Update the phone number environment variable with your phone number.
 */

const AWS = require('aws-sdk');

const phone_number = process.env.phone_number;
const SNS = new AWS.SNS({ apiVersion: '2010-03-31' });

exports.handler = (event, context, callback) => {
    console.log('Received event:', event);

    // publish message
    const params = {
        Message: 'Your DeepLens device just identified a person. Congratulations!',
        PhoneNumber: phone_number
    };
    if (event.Hotdog > 0.50)
        SNS.publish(params, callback);
};
```

3. Click Save

# Step 6- Disable the IoT rule

1. Go to IoT console (https://console.aws.amazon.com/iotv2/home?region=us-east-1#) and choose Act

2. Find the rule that you just created and disable it (Unless you want DeepLens to keep pinging you when it detects a person throughout the day ;) )
