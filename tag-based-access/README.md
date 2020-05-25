## Overview

This example is deliberately simple in many aspects and does a few things that you probably don't want to do at scale (like hardcoding role names). 

This example creates 2 resources:

|Resource|Description|
|---|---|
|PipelineExecutionRole|The IAM role that developers work under. 
|PipelineExecutionRolePolicy|The IAM policy attached to the above role. 



## Links

https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html

The following link needs Authorisaion based on tags
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html

## Deployment with AWSCLI 

Update you ~/.aws/config so you can assume role easily. I've explicitly named the role in this example to make this stuff easier to get going.

```
[profile execution-role]
role_arn = arn:aws:iam::726508711480:role/ExecutionRole
source_profile = default
```

**Update the Parameters file!**



Execution Role Stack
```
aws cloudformation create-stack --stack-name IAM-Tag-Based-Permissions-Example --template-body file://cloudformation/iam-execution-role.yml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM 

aws cloudformation update-stack --stack-name IAM-Tag-Based-Permissions-Example  --template-body file://cloudformation/iam-execution-role.yml --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM

aws cloudformation delete-stack --stack-name IAM-Tag-Based-Permissions-Example
```



Application Stack (ADMIN)
```
aws cloudformation create-stack --stack-name My-Application-Example --template-body file://cloudformation/example-application.yml 

aws cloudformation create-stack --stack-name My-Application-Example --template-body file://cloudformation/example-application.yml 

aws cloudformation delete-stack --stack-name My-Application-Example
```
Application Stack
```
aws cloudformation create-stack --profile execution-role --stack-name My-Application-Example --template-body file://cloudformation/example-application.yml 

aws cloudformation update-stack --profile execution-role --stack-name My-Application-Example --template-body file://cloudformation/example-application.yml 

aws cloudformation delete-stack --profile execution-role --stack-name My-Application-Example 

```

