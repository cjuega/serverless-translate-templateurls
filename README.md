# Serverless Translate TemplateURLs
Small plugin for Serverless Framework to package AWS CloudFormation templates with local references.

serverless-translate-templateurls plugin uses aws-cli [package](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/package.html) to create local artifacts, upload them to AWS S3; and create a copy in which local references are replaced by S3 references. This allows to split (and reuse) CloudFormation templates in smaller logical stacks. Without this plugin (or similar mechanism), your calls to `serverless deploy` will fail because AWS CloudFormation service can't access your local files.

## Prerequisites
* node (>=10.x).
* serverless framework installed (>=1.50.1) ([Serverless Framework](https://serverless.com/framework/docs/getting-started/)).
* aws-cli must be installed and configured ([AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)).

## Installation
```sh
npm i serverless-translate-templateurls
```

## Full Example

Within your `serverless.xml`

```yml
custom:
  serverless-translate-templateurls:
    s3Bucket: my-setup-bucket
    inFolder: cf-templates
    outFolder: resources
plugins:
  - serverless-translate-templateurls
```

A template with local references within `./cf-templates/`:
```yml
Resources:
  SQSQueue:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: ../aws-cf-templates/messaging/sqs.yml # local path to a nested CloudFormation template
      Parameters:
        QueueName: my-sqs-queue
        MessageRetentionPeriod: 14400
        ReceiveMessageWaitTimeSeconds: 20
        VisibilityTimeout: 3600
        MaxReceiveCount: 2
```

Then, calling to `serverless translate` will create the following template at `./resources/`:

```yml
Resources:
  SQSQueue:
    Properties:
      Parameters:
        MaxReceiveCount: 2
        MessageRetentionPeriod: 14400
        ReceiveMessageWaitTimeSeconds: 20
        VisibilityTimeout: 3600
      TemplateURL: https://s3.amazonaws.com/my-setup-bucket/serverless-translate-templateurls/0fc5db4d1d1fbfb2a6954773654c74fa.template
    Type: AWS::CloudFormation::Stack
```

## Usage
Hooks to `serverless package` and `serverless deploy` are included, so this plugin should work out-of-the-box. However it can be customized with some options. Also, a new serverless command is included: [translate](#Translate).

### Custom options
> s3Bucket

AWS S3 bucket name to be used when uploading the artifacts that are referenced in your templates.

If it is not specified, then the S3 bucket from Serverless Framework will be used.

_NOTE: using Serverless' bucket may incur in a race condition when the Serverless project hasn't created its bucket yet! Basically the plugin will try to use a bucket that doesn't exist yet_

> inFolder

local folder where this plugin will look for CloudFormation templates to package.

If it is not specified, then it uses `cf-templates` as folder.

> outFolder

local folder where this plugin will output CloudFormation templates with local references changed by remote ones.

If it is not specified, then it uses `resources` as folder.

### Translate
Translate command performs the actual conversion. It's a convenient command to replace local references by S3 references outside Serverless' lifecycle.
```sh
serverless translate
```

## License
MIT