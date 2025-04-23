# root user login notification-aws

## Prerequisities
- Enable CloudTrail

## Plan
In a nutshell, need to define a CloudWatch Events rule to detect the login action and send an SNS notification. The trick is to use the us-east-1 region, as that is where the root users log in. Setting up the Event Rule in that region is essential.

## Step 1: Set up an SNS topic
First, switch to the us-east-1 region before doing anything else. Then create an SNS topic, subscribe to it, then confirm the subscription.

Unfortunately, the default topic policy does not allow CloudWatch Events to publish to the topic. To allow it, replace the topic policy with this one:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "<arn of the topic>"
    }
  ]
}
```
## Step 2: Create an Event Rule
The event is called AWS Console Sign In via CloudTrail. Still in the us-east-1 region, go to the CloudWatch console, and under Events and Rules, add a new rule.

For the Event pattern, use this:
```
{
  "detail-type": [
    "AWS Console Sign In via CloudTrail"
  ],
  "detail": {
    "userIdentity": {
      "type": [
        "Root"
      ]
    }
  }
}
```
Add the SNS topic as the target.

## Step 3: Test it
Log out and log back in with the root user. An email will arrive seconds after.

## CloudFormation
Deploy the CloudFormation template to the **us-east-1** region and subscribe to the resulting topic.
The template to have the notification set up for you automatically. Just don't forget to switch to the us-east-1 region before deploying it, and subscribe to the resulting SNS topic after.
