# GrassFormation

GrassFormation is a collection of AWS lambda functions that allows you to deploy Greengrass resources with CloudFormation that are otherwise not supported.

## Installing

Deploy the CloudFormation custom resource handler lambda functions with [serverless](https://serverless.com/framework/docs/) framework.

Now you can start writing CloudFormation stacks that deploy Greengrass resources. An example template is provided in the `test/fullgrass.yaml` file.

## Deploying the sample CloudFormation stack

Pick up `test/fullgrass.yaml` and head to the [CloudFormation management console](https://console.aws.amazon.com/cloudformation/home). Select "Create Stack" and upload the template file to S3. You will have to fill out the following stack parameters:

 - `CSRParameter`: A Certificate Signing Request, created along with the certificates that you will deploy on your Greengrass Core. For more information check the [AWS IoT Documentation](https://docs.aws.amazon.com/iot/latest/apireference/API_CreateCertificateFromCsr.html). It should have the following format:

```
-----BEGIN CERTIFICATE REQUEST-----
[base64 encoded certificate request]
-----END CERTIFICATE REQUEST-----
```

 - `GroupNameParameter`: The name of the Greengrass Group.
 - `CoreShadowHandlerARN`: The ARN of the lambda function that will handle the Core Shadow updates.
 - `GFStackName`: The name of the CloudFormation stack that deployed the custom resource handler lambda functions during installation. For example if you used the `dev` stage and did not modify the stack name in `serverless.yml` then the value of this parameter will be `grassformation-dev`.

## Deleting the sample CloudFormation stack

There is one glitch when you want to delete the sample CloudFormation stack. The `AWS::IoT::Certificate` type resource named `CoreCert` is left in `ACTIVATE` state after the deployment of the stack. This is necessary for the Greengrass Core to work. However this resource can not be deleted until it is not deactivated. So before starting the deletion of the stack you should deactivate it manually. Pick up the Physical ID of the `CoreCert` resource: you can find it under the "Resources" section of the details page of your deployed CloudFormation stack. Then call this command of the aws cli:

```
aws iot update-certificate --certificate-id [Physical ID from CF] --new-status INACTIVE
```

Now you can delete the deployed stack.

## Supported parameters

All custom resource lambdas pass most of their attributes to the appropriate AWS Greengrass API. However all functions abstract away the concept of "resource version" of Greengrass. Whenever you update your CloudFormation stack with GreenFormation, a new version of the updated entity will be automatically created. The attributes of the main entity (currently only `Name` for both entities) and those ones of the appropriate version entity are merged. Bearing this in mind you can find a list of the supported attributes below.

### GrassFormationGroup

Supported attributes:
 - `Name` (string): The name of the Greengrass Group
 - `GroupRoleArn` (string): The ARN of the IAM Role to be associated with the Group. Your AWS Greengrass core will use the role to access AWS cloud services. The role's permissions should allow Greengrass core Lambda functions to perform actions against the cloud.
 - `CoreDefinitionVersionArn`, `DeviceDefinitionVersionArn`, `FunctionDefinitionVersionArn`, `SubscriptionDefinitionVersionArn`, `LoggerDefinitionVersionArn`, `ResourceDefinitionVersionArn`: see [Greengrass CreateGroupVersion API](https://docs.aws.amazon.com/greengrass/latest/apireference/creategroupversion-post.html) for more info.

### GrassFormationCore

Supported attributes:
 - `Name` (string): The name of the Greengrass Core
 - `Cores`: see [Greengrass CreateCoreDefinitionVersion API](https://docs.aws.amazon.com/greengrass/latest/apireference/createcoredefinitionversion-post.html) for more info.

### GrassFormationResource

Supported attributes:
 - `Name` (string): The name of the Greengrass Resources definition
 - `Resources`: see [Greengrass CreateResourceDefinitionVersion API](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinitionversion-post.html) for more info.

## Returned values

Similarly to Supported Parameters, the custom resource lambda functions return pretty much whatever the appropriate AWS API returns. You can find an example how to use them in the `GreengrassGroup` resource definition in the sample stack:

```
CoreDefinitionVersionArn: !GetAtt CoreDefinition.LatestVersionArn
```
