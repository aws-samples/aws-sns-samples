# Amazon Simple Notification Service (SNS) - Message Data Protection

[Amazon SNS](https://aws.amazon.com/sns/) is a fully managed serverless messaging service. Amazon SNS provides topics for push-based, many-to-many, pub/sub messaging for decoupling distributed systems, microservices, and event-driven serverless applications. As applications grow, so does the amount of data transmitted and the number of systems sending (publishers) and receiving (subscribers) data. When moving data between different applications, it is important to have guardrails in place to help you comply with data privacy regulations that require you to safeguard sensitive personally identifiable information (PII) and protected health information (PHI).

With message data protection for Amazon SNS, you can scan messages in real-time for PII/PHI, and receive audit reports that contain the results of your scan. Also, you can prevent applications from receiving sensitive data by masking or redacting sensitive data in the message payload or blocking messages entirely flowing either inbound to a topic or outbound to a subscription. Message data protection supports a repository of several unique PII/PHI data identifiers, such as name, address, social security number, credit card number, and prescription drug code. These capabilities can help you adhere to a variety of compliance regulations, including HIPAA, FedRAMP, GDPR, and PCI. You can also define your own custom regexes to scan for business-specific sensitive data, using [Custom Data Identifiers](https://docs.aws.amazon.com/sns/latest/dg/sns-message-data-protection-custom-data-identifiers.html). For more information, see [Message Data Protection](https://docs.aws.amazon.com/sns/latest/dg/message-data-protection.html), in the Amazon SNS Developer Guide.

This repository contains the [AWS CloudFormation](https://aws.amazon.com/cloudformation) templates to deploy the example use case, as described in the [Mask and redact sensitive data published to Amazon SNS using managed and custom data identifiers](https://aws.amazon.com/blogs/compute/mask-and-redact-sensitive-data-published-to-amazon-sns-using-managed-and-custom-data-identifiers/) blog post.

## Dependencies

* [AWS CLI](https://aws.amazon.com/cli/)
* [AWS SAM Toolkit](https://aws.amazon.com/serverless/sam/)

## Installation

There are two ways to set up this example. One option is to directly run the `deploy` script, which automatically executes the four SAM templates in scope. Another option is to execute each SAM template individually.

### Option 1 - Run the aggregate script

Run the following command on your terminal. 

```shell
./deploy MyDataProtectionDemo
```

### Option 2 - Run the individual templates

1. First, run the following command to deploy the pre-requisites stack to create two IAM roles, which we will use in subsequent steps. This first step outputs the name of both IAM roles: `PaymentRoleExport` and `ReportingRoleExport`:

```shell
sam deploy --resolve-s3 --stack-name approval-topic-prerequisites --capabilities CAPABILITY_IAM --template 0-prereqs.yml
```

2. Then, run the following command to deploy the stack for the Amazon SNS topic owner: 

```shell
sam deploy --resolve-s3 --stack-name approval-topic --capabilities CAPABILITY_IAM --template 1-topicowner.yml
```

3. Next, gather the name of both IAM roles as we will use them to set up the Amazon SNS subscriber stacks below:

```shell
PAYMENT_ROLE_OUTPUT_KEY='PaymentRoleExport'
REPORTING_ROLE_OUTPUT_KEY='ReportingRoleExport'

payment_role=$(aws cloudformation describe-stacks --stack-name approval-topic-prerequisites --output text --query "Stacks[].Outputs[?OutputKey==\`$PAYMENT_ROLE_OUTPUT_KEY\`].OutputValue")
reporting_role=$(aws cloudformation describe-stacks --stack-name approval-topic-prerequisites --output text --query "Stacks[].Outputs[?OutputKey==\`$REPORTING_ROLE_OUTPUT_KEY\`].OutputValue")
```

4. Next, create the Amazon SNS subscriber stack for Payment. We run this deployment using the IAM role that we created for the Payment system, in our use case:

```shell
sam deploy --resolve-s3 --stack-name payment-subscription --capabilities CAPABILITY_IAM --template 2.1-payment.yml --role-arn $payment_role
```

5. Lastly, create the Amazon SNS subscriber stack for Reporting. We run this deployment using the IAM role that we created for the Reporting system, in our use case:

```shell
sam deploy --resolve-s3 --stack-name reporting-subscription --capabilities CAPABILITY_IAM --template 2.2-reporting.yml --role-arn $reporting_role
```

## Testing

Now that you have deployed all AWS CloudFormation stacks needed, you may go back to the [Mask and redact sensitive data published to Amazon SNS using managed and custom data identifiers](https://aws.amazon.com/blogs/compute/mask-and-redact-sensitive-data-published-to-amazon-sns-using-managed-and-custom-data-identifiers/) blog post, and follow the steps to test the message data protection feature of Amazon SNS.
