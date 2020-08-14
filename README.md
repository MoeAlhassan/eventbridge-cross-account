# Cross-Account Events to Custom Bus

These repo contains two CloudFormation templates for sending events from the default event bus in one account (sender account) to a custom event bus in another account (receiver account). The receiver account will log the events from the sender account to a CloudWatch log group. 

- [Cross-Account Events to Custom Bus](#cross-account-events-to-custom-bus)
  - [Directory](#directory)
  - [Usage](#usage)
    - [Step 1: Get AccountID of Sender Account](#step-1-get-accountid-of-sender-account)
    - [Step 2: Create Stack for Receiver Account](#step-2-create-stack-for-receiver-account)
    - [Step 3: Get Custom Bus ARN of the Receiver](#step-3-get-custom-bus-arn-of-the-receiver)
    - [Step 4: Create Stack for Sender Account](#step-4-create-stack-for-sender-account)
    - [Step 5: Check for Events](#step-5-check-for-events)

## Directory

This repo contains: 
- `/cross-account-events/cross-account-receiver.yaml` | CloudFormation for the account **RECEIVING** events
- `/cross-account-events/cross-account-sender.yaml` | CloudFormation for the account **SENDING** events
- `/cross-account-events/README.md` | You are here

## Usage

### Step 1: Get AccountID of Sender Account

You'll need the AWS AccountID of the Sender Account so the template can set up:
- An event bus policy that allows the sender account to `PutEvents` to the receiver account
- An event pattern that looks for events from the sender account
  - ex: `{"account": ["SenderAccountID"]}`

### Step 2: Create Stack for Receiver Account

Create the resources used to receive events. Using the CLI: 

```aws cloudformation create-stack --stack-name **NameForYourStack** --template-body file://cross-account-receiver.yaml --capabilities "CAPABILITY_NAMED_IAM" --parameters ParameterKey=SenderAccountID,ParameterValue=**SenderAccountID** ParameterKey=CustomBusName,ParameterValue=**CustomBusName**```

### Step 3: Get Custom Bus ARN of the Receiver 

Get the Custom Bus ARN to use in the next stack. 

```aws cloudformation describe-stacks --stack-name **StackNameFromStep2** ```

The Custom Bus ARN will be under `Outputs` listed as `OutputValue` 

### Step 4: Create Stack for Sender Account

Create the resources used to send events. Using the CLI: 
```aws cloudformation create-stack --stack-name **NameForYourStack**  --template-body file://cross-account-sender.yaml --capabilities "CAPABILITY_NAMED_IAM" --parameters ParameterKey=CustomEventBusArn,ParameterValue=**CustomBusArnFromStep3```

### Step 5: Check for Events

Once both stacks are in a `CREATE_COMPLETE` state, check the CloudWatch log group in the receiver account and there should be log streams that illustrate cross-account events. 


