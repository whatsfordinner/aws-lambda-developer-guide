# Using AWS Lambda environment variables<a name="configuration-envvars"></a>

You can use environment variables to adjust your function's behavior without updating code\. An environment variable is a pair of strings that is stored in a function's version\-specific configuration\. The Lambda runtime makes environment variables available to your code and sets additional environment variables that contain information about the function and invocation request\.

**Note**  
To increase database security, we recommend that you use AWS Secrets Manager instead of environment variables to store database credentials\. For more information, see [Configuring database access for a Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/configuration-database.html)\.

Environment variables are not evaluated prior to the function invocation\. Any value you define is considered a literal string and not expanded\. Perform the variable evaluation in your function code\.

**Topics**
+ [Configuring environment variables](#configuration-envvars-config)
+ [Configuring environment variables with the API](#configuration-envvars-api)
+ [Example scenario for environment variables](#configuration-envvars-example)
+ [Retrieve environment variables](#configuration-envvars-retrieve)
+ [Defined runtime environment variables](#configuration-envvars-runtime)
+ [Securing environment variables](#configuration-envvars-encryption)
+ [Sample code and templates](#configuration-envvars-samples)

## Configuring environment variables<a name="configuration-envvars-config"></a>

You define environment variables on the unpublished version of your function\. When you publish a version, the environment variables are locked for that version along with other [version\-specific configuration](configuration-function-common.md)\. 

You create an environment variable for your function by defining a key and a value\. Your function uses the name of the key to retrieve the value of environment variable\.

**To set environment variables in the Lambda console**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. Choose a function\.

1. Choose **Configuration**, then choose **Environment variables**\.

1. Under **Environment variables**, choose **Edit**\.

1. Choose **Add environment variable**\.

1. Enter a key and value\.

**Requirements**
   + Keys start with a letter and are at least two characters\.
   + Keys only contain letters, numbers, and the underscore character \(`_`\)\.
   + Keys aren't [reserved by Lambda](#configuration-envvars-runtime)\.
   + The total size of all environment variables doesn't exceed 4 KB\.

1. Choose **Save**\.

## Configuring environment variables with the API<a name="configuration-envvars-api"></a>

To manage environment variables with the AWS CLI or AWS SDK, use the following API operations\.
+ [UpdateFunctionConfiguration](API_UpdateFunctionConfiguration.md)
+ [GetFunctionConfiguration](API_GetFunctionConfiguration.md)
+ [CreateFunction](API_CreateFunction.md)

The following example sets two environment variables on a function named `my-function`\.

```
aws lambda update-function-configuration --function-name my-function \
    --environment "Variables={BUCKET=my-bucket,KEY=file.txt}"
```

When you apply environment variables with the `update-function-configuration` command, the entire contents of the `Variables` structure is replaced\. To retain existing environment variables when you add a new one, include all existing values in your request\.

To get the current configuration, use the `get-function-configuration` command\.

```
aws lambda get-function-configuration --function-name my-function
```

You should see the following output:

```
{
    "FunctionName": "my-function",
    "FunctionArn": "arn:aws:lambda:us-east-2:123456789012:function:my-function",
    "Runtime": "nodejs12.x",
    "Role": "arn:aws:iam::123456789012:role/lambda-role",
    "Environment": {
        "Variables": {
            "BUCKET": "my-bucket",
            "KEY": "file.txt"
        }
    },
    "RevisionId": "0894d3c1-2a3d-4d48-bf7f-abade99f3c15",
    ...
}
```

To ensure that the values don't change between when you read the configuration and when you update it, you can pass the revision ID from the output of `get-function-configuration` as a parameter to `update-function-configuration`\.

To configure a function's encryption key, set the `KMSKeyARN` option\.

```
aws lambda update-function-configuration --function-name my-function \
   --kms-key-arn arn:aws:kms:us-east-2:123456789012:key/055efbb4-xmpl-4336-ba9c-538c7d31f599
```

## Example scenario for environment variables<a name="configuration-envvars-example"></a>

You can use environment variables to customize function behavior in your test environment and production environment\. For example, you can create two functions with the same code but different configurations\. One function connects to a test database, and the other connects to a production database\. In this situation, you use environment variables to tell the function the hostname and other connection details for the database\. 

The following example shows how to define the database host and database name as environment variables\.

![\[\]](http://docs.aws.amazon.com/lambda/latest/dg/images/console-env.png)

If you want your test environment to generate more debug information than the production environment, you could set an environment variable to configure your test environment to use more verbose logging or more detailed tracing\.

## Retrieve environment variables<a name="configuration-envvars-retrieve"></a>

To retrieve environment variables in your function code, use the standard method for your programming language\.

------
#### [ Node\.js ]

```
let region = process.env.AWS_REGION
```

------
#### [ Python ]

```
import os
  region = os.environ['AWS_REGION']
```

**Note**  
In some cases, you may need to use the following format:  

```
region = os.environ.get('AWS_REGION')
```

------
#### [ Ruby ]

```
region = ENV["AWS_REGION"]
```

------
#### [ Java ]

```
String region = System.getenv("AWS_REGION");
```

------
#### [ Go ]

```
var region = os.Getenv("AWS_REGION")
```

------
#### [ C\# ]

```
string region = Environment.GetEnvironmentVariable("AWS_REGION");
```

------
#### [ PowerShell ]

```
$region = $env:AWS_REGION
```

------

Lambda stores environment variables securely by encrypting them at rest\. You can [configure Lambda to use a different encryption key](#configuration-envvars-encryption), encrypt environment variable values on the client side, or set environment variables in an AWS CloudFormation template with AWS Secrets Manager\.

## Defined runtime environment variables<a name="configuration-envvars-runtime"></a>

Lambda [runtimes](lambda-runtimes.md) set several environment variables during initialization\. Most of the environment variables provide information about the function or runtime\. The keys for these environment variables are *reserved* and cannot be set in your function configuration\.

**Reserved environment variables**
+ `_HANDLER` – The handler location configured on the function\.
+ `_X_AMZN_TRACE_ID` – The [X\-Ray tracing header](services-xray.md)\.
+ `AWS_REGION` – The AWS Region where the Lambda function is executed\.
+ `AWS_EXECUTION_ENV` – The [runtime identifier](lambda-runtimes.md), prefixed by `AWS_Lambda_`—for example, `AWS_Lambda_java8`\.
+ `AWS_LAMBDA_FUNCTION_NAME` – The name of the function\.
+ `AWS_LAMBDA_FUNCTION_MEMORY_SIZE` – The amount of memory available to the function in MB\.
+ `AWS_LAMBDA_FUNCTION_VERSION` – The version of the function being executed\.

  `AWS_LAMBDA_INITIALIZATION_TYPE` – The initialization type of the function, which is either `on-demand` or `provisioned-concurrency`\. For information, see [ Configuring provisioned concurrency](provisioned-concurrency.md)\. 
+ `AWS_LAMBDA_LOG_GROUP_NAME`, `AWS_LAMBDA_LOG_STREAM_NAME` – The name of the Amazon CloudWatch Logs group and stream for the function\.
+ `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` – The access keys obtained from the function's [execution role](lambda-intro-execution-role.md)\.
+ `AWS_LAMBDA_RUNTIME_API` – \([Custom runtime](runtimes-custom.md)\) The host and port of the [runtime API](runtimes-api.md)\.
+ `LAMBDA_TASK_ROOT` – The path to your Lambda function code\.
+ `LAMBDA_RUNTIME_DIR` – The path to runtime libraries\.
+ `TZ` – The environment's time zone \(`UTC`\)\. The execution environment uses NTP to synchronize the system clock\.

The following additional environment variables aren't reserved and can be extended in your function configuration\.

**Unreserved environment variables**
+ `LANG` – The locale of the runtime \(`en_US.UTF-8`\)\.
+ `PATH` – The execution path \(`/usr/local/bin:/usr/bin/:/bin:/opt/bin`\)\.
+ `LD_LIBRARY_PATH` – The system library path \(`/lib64:/usr/lib64:$LAMBDA_RUNTIME_DIR:$LAMBDA_RUNTIME_DIR/lib:$LAMBDA_TASK_ROOT:$LAMBDA_TASK_ROOT/lib:/opt/lib`\)\.
+ `NODE_PATH` – \([Node\.js](lambda-nodejs.md)\) The Node\.js library path \(`/opt/nodejs/node12/node_modules/:/opt/nodejs/node_modules:$LAMBDA_RUNTIME_DIR/node_modules`\)\.
+ `PYTHONPATH` – \([Python 2\.7, 3\.6, 3\.8](lambda-python.md)\) The Python library path \(`$LAMBDA_RUNTIME_DIR`\)\.
+ `GEM_PATH` – \([Ruby](lambda-ruby.md)\) The Ruby library path \(`$LAMBDA_TASK_ROOT/vendor/bundle/ruby/2.5.0:/opt/ruby/gems/2.5.0`\)\.
+ `AWS_XRAY_CONTEXT_MISSING` – For X\-Ray tracing, Lambda sets this to `LOG_ERROR` to avoid throwing runtime errors from the X\-Ray SDK\.
+ `AWS_XRAY_DAEMON_ADDRESS` – For X\-Ray tracing, the IP address and port of the X\-Ray daemon\.
+ `AWS_LAMBDA_DOTNET_PREJIT` – For the \.NET 3\.1 runtime, set this variable to enable or disable \.NET 3\.1 specific runtime optimizations\. Values include `always`, `never`, and `provisioned-concurrency`\. For information, see [ Configuring provisioned concurrency](provisioned-concurrency.md)\.

The sample values shown reflect the latest runtimes\. The presence of specific variables or their values can vary on earlier runtimes\.

## Securing environment variables<a name="configuration-envvars-encryption"></a>

Lambda encrypts environment variables with a key that it creates in your account \(an AWS managed customer master key \(CMK\)\)\. Use of this key is free\. You can also choose to provide your own key for Lambda to use instead of the default key\.

**Warning**
If you use the default key then Lambda creates a grant on it for the function's execution role when an environment variable is configured for the first time\. If the execution role is deleted and then re-created then this grant will no longer be valid and the function will be unable to run\. You will not be able to re-create this grant without deleting and recreating the function\.

When you provide the key, only users in your account with access to the key can view or manage environment variables on the function\. Your organization might also have internal or external requirements to manage keys that are used for encryption and to control when they're rotated\.

**To use a customer managed CMK**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. Choose a function\.

1. Choose **Configuration**, then choose **Environment variables**\.

1. Under **Environment variables**, choose **Edit**\.

1. Expand **Encryption configuration**\.

1. Choose **Use a customer master key**\.

1. Choose your customer managed CMK\.

1. Choose **Save**\.

Customer managed CMKs incur standard [AWS KMS charges](https://aws.amazon.com/kms/pricing/)\.

No AWS KMS permissions are required for your user or the function's execution role to use the default encryption key\. To use a customer managed CMK, you need permission to use the key\. Lambda uses your permissions to create a grant on the key\. This allows Lambda to use it for encryption\.
+ `kms:ListAliases` – To view keys in the Lambda console\.
+ `kms:CreateGrant`, `kms:Encrypt` – To configure a customer managed CMK on a function\.
+ `kms:Decrypt` – To view and manage environment variables that are encrypted with a customer managed CMK\.

You can get these permissions from your user account or from a key's resource\-based permissions policy\. `ListAliases` is provided by the [managed policies for Lambda](access-control-identity-based.md)\. Key policies grant the remaining permissions to users in the **Key users** group\.

Users without `Decrypt` permissions can still manage functions, but they can't view environment variables or manage them in the Lambda console\. To prevent a user from viewing environment variables, add a statement to the user's permissions that denies access to the default key, a customer managed key, or all keys\.

**Example IAM policy – Deny access by key ARN**  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Deny",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": "arn:aws:kms:us-east-2:123456789012:key/3be10e2d-xmpl-4be4-bc9d-0405a71945cc"
        }
    ]
}
```

![\[\]](http://docs.aws.amazon.com/lambda/latest/dg/images/env-accessdenied.png)

For details on managing key permissions, see [Using key policies in AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)\.

You can also encrypt environment variable values on the client side before sending them to Lambda, and decrypt them in your function code\. This obscures secret values in the Lambda console and API output, even for users who have permission to use the key\. In your code, you retrieve the encrypted value from the environment and decrypt it by using the AWS KMS API\.

**To encrypt environment variables on the client side**

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) on the Lambda console\.

1. Choose a function\.

1. Choose **Configuration**, then choose **Environment variables**\.

1. Under **Environment variables**, choose **Edit**\.

1. Expand **Encryption configuration**\.

1. Choose **Enable helpers for encryption in transit**\.

1. Choose **Encrypt** next to a variable to encrypt its value\.

1. Choose **Save**\.

**Note**  
When you use the console encryption helpers, your function needs permission to call the `kms:Decrypt` API operation in its [execution role](lambda-intro-execution-role.md)\.

To view sample code for your function's language, choose **Code** next to an environment variable\. The sample code shows how to retrieve an environment variable in a function and decrypt its value\.

Another option is to store passwords in AWS Secrets Manager secrets\. You can reference the secret in your AWS CloudFormation templates to set passwords on databases\. You can also set the value of an environment variable on the Lambda function\. For an example, see the next section\.

## Sample code and templates<a name="configuration-envvars-samples"></a>

Sample applications in this guide's GitHub repository demonstrate the use of environment variables in function code and AWS CloudFormation templates\.

**Sample applications**
+ [Blank function](samples-blank.md) – Create a function and an Amazon SNS topic in the same template\. Pass the name of the topic to the function in an environment variable\. Read environment variables in code \(multiple languages\)\.
+ [RDS MySQL](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/rds-mysql) – Create a VPC and an Amazon RDS DB instance in one template, with a password stored in Secrets Manager\. In the application template, import database details from the VPC stack, read the password from Secrets Manager, and pass all connection configuration to the function in environment variables\.