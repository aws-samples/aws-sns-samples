## Amazon Simple Notification Service (SNS) - Message Data Protection

SNS is a fully managed serverless messaging service. SNS provides topics for push-based, many-to-many pub/sub messaging for decoupling distributed systems, microservices, and event-driven serverless applications. As applications grow, so does the amount of data transmitted and the number of systems sending (publishers) and receiving (subscribers) data. When moving data between different applications, it is important to have guardrails in place to help you comply with data privacy regulations that require you to safeguard sensitive personally identifiable information (PII) or personal health information (PHI).

With message data protection, you can scan messages in real-time for PII/PHI and receive audit reports that contain the results of your scan and prevent applications from receiving the sensitive data by blocking messages flowing inbound to a topic or outbound to a subscription. Message data protection supports a repository of over 25 unique PII/PHI data identifiers, such as people’s names, addresses, social security numbers, credit card numbers, and prescription drug codes. These capabilities can help you adhere to a variety of compliance regulations, including HIPAA, FedRAMP, GDPR, and PCI. For more information, including the complete list of supported data identifiers, see Message Data Protection, in the SNS Developer Guide.

## Prerequisites
* Verify [AWS CLI](https://aws.amazon.com/cli/) is installed
* Verify [SAM Toolkit](https://aws.amazon.com/serverless/sam/) is installed

## Installation

There are two ways to setup this example. One option is to directly run the `deploy` script, which automatically executes the four SAM templates in scope. Another option is to execute each SAM template individually.

### Option 1 - Aggregate script

Run the following command on your terminal. 

```shell
./deploy MyDataProtectionDemo
```

### Option 2 - Individual templates

1. First, run the Pre-requisites stack that will create two IAM roles that will be used in subsequent steps:
```shell
sam deploy --resolve-s3 --stack-name clinic-prerequisites --capabilities CAPABILITY_IAM --template 0-prereqs.yml
```
This step should output the names of two IAM roles under the name `BillingRoleExport` and `SchedulingRoleExport`

2. Deploy the stack for topic-owner
```shell
sam deploy --resolve-s3 --stack-name clinic-topic --capabilities CAPABILITY_IAM --template 1-topicowner.yml
```

3. Gather the names of the two IAM roles
```shell
BILLING_ROLE_OUTPUT_KEY='BillingRoleExport'
SCHEDULING_ROLE_OUTPUT_KEY='SchedulingRoleExport'

billing_role=$(aws cloudformation describe-stacks --stack-name clinic-prerequisites --output text --query "Stacks[].Outputs[?OutputKey==\`$BILLING_ROLE_OUTPUT_KEY\`].OutputValue")
scheduling_role=$(aws cloudformation describe-stacks --stack-name clinic-prerequisites --output text --query "Stacks[].Outputs[?OutputKey==\`$SCHEDULING_ROLE_OUTPUT_KEY\`].OutputValue")
```

4. Create the subscriber stack for Scheduling
```shell
sam deploy --resolve-s3 --stack-name clinic-scheduling --capabilities CAPABILITY_IAM --template 2.1-scheduling.yml --role-arn $scheduling_role
```

5. Create the subscriber stack for Billing
```shell
sam deploy --resolve-s3 --stack-name clinic-billing --capabilities CAPABILITY_IAM --template 2.2-billing.yml --role-arn $billing_role
```

