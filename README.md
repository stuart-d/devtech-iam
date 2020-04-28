# devtech-iam

This repo demonstrates how to use AWS IAM Permisisons Boundaries as a mechanism for allowing teams to create their own IAM roles while maintaining control of the scope of permissions on those roles.

## Overview

This template creates 3 resources:

|Resource|Description|
|---|---|
|DeveloperRole|The IAM role that developers work under. 
|DeveloperRolePolicy|The IAM policy attached to the above role. This policy has conditional permissions to create additional IAM resources.
|PermissionBoundaryPolicyForRolesCreatedByDeveloper| The IAM policy that defines the permissions boundary for roles created by the DeveloperRole.
|

In this example:

A Developer can create an IAM role as long as they satisfy two conditions:

1) The role name must conform to the pattern defined by the CreatedResourcesPattern parameter.
2) They must reference the permissions boundary policy created in this template.

Additonally, if they are creating an IAM policy:

3) The policy name must conform to the pattern defined by the CreatedResourcesPattern parameter.

Regardless of the policies that get attatched to the role that DeveloperRole creates, the PermissionBoundaryPolicyForRolesCreatedByDeveloper with ensure that the effective permisisons are not greater than s3:* in ap-southeast-2




## Notes and FAQ

### How does this actually restrict access?

Its probably best to read the AWS doco in full, but in short: DeveloperRole can only create and delete roles and policies that match the pattern defined by the parameter CreatedResourcesPattern. From a policy permissions perspective, although DeveloperRole can create (or select an existing) policy with unlimited scope, they cannot attach that policy to a role without supplying the name of the permission boundary policy. The permission boundary policy is evaluated against the supplied policy to form the effective permissions.

### Be careful with assigning the Developer role increased IAM permissions

In IAM, an explicit deny always trumps any allow. So you might think its ok to assign DeveloperRole the managed policy AdministratorAccess or iam:* with the thought that the more granular IAM permissions defined in these templates will override the *. This is not the case, as this policy defines conditional allows. If you explicitly give DeveloperRole more IAM permissions than this template defines, then those will be the effective permissions.

### Using named resources with roles and policies may cause challenges

This example uses RoleName and other similar properties when creating resources. This is great because you can be sure the resource is named exactly as you wanted, however this can be troublesome in the event that CloudFomration needs to delete a resource to replace it. You might have to duplicate the resource with a new name before you can delete. Just be aware of this.

### Is the CreatedRolesPattern case sensitive?

Yes.

### What if I want it to not be NOT case sensitive?

You'll need to configure multiple resources entries in each releavant IAM policy:

e.g change:
```
   Resource: 
    - !Sub arn:aws:iam::${AWS::AccountId}:role/${CreatedRolesPattern}
```
To:
```
   Resource: 
    - !Sub arn:aws:iam::${AWS::AccountId}:role/${CreatedRolesPattern}
    - !Sub arn:aws:iam::${AWS::AccountId}:role/${CreatedRolesPattern2}      
```
This may become painful using parameters. 

### Why have you split up permissions like CreateRole and DeleteRole between SIDs?

This one's a bit tricky. Essentially, the API's that create and update resources accept both the pattern conditional in the ARN and a PermissionsBoundary attribute. The delete and list methods accept the pattern conditional in the ARN but DON'T accept permissions boudaries.

So for something like "iam:DeleteRole" if you list it in the SID with iam:CreateRole (where the policy requires both ARN and PermissionsBoundary), you end up in a situation where IAM internal logic expect you to satisfy both rquirements but the IAM API won't accept the permission boundary in the request. So you can never delete your own resources.

Another way of thinking about it is the API doesn't expect the PermissionBoundary property when it's not creating resources.

### So how does the developer pass the PermisisonBoundary attribute when creating roles?

**Console**

In the console, there is a field when creating the Role that allows you to enter it.

**AWS CLI**

<pre>
create-role
[--path <value>]
--role-name <value>
--assume-role-policy-document <value>
[--description <value>]
[--max-session-duration <value>]
<b>[--permissions-boundary <value>]</b>
[--tags <value>]
[--cli-input-json <value>]
[--generate-cli-skeleton <value>]
</pre>

https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html

**CloudFormation**

In CloudFormation, you include it as a properterty:

  <pre>
Type: AWS::IAM::Role
Properties: 
  AssumeRolePolicyDocument: Json
  Description: String
  ManagedPolicyArns: 
    - String
  MaxSessionDuration: Integer
  Path: String
  <b>PermissionsBoundary: String</b>
  Policies: 
    - Policy
  RoleName: String
  Tags: 
    - Tag
  </pre>

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

### What do errors look like?

If the dev role tries to create a role that doesn't match the pattern OR doesn't supply the name of the Permissions Boundary role, they get the same error, example below. In this example my role name "MyCoolDevRole" didn't match the pattern.

```
User: arn:aws:sts::123456789:assumed-role/iam-perm-boundary-DeveloperRole-14W1CA3SPJ6FS/sdevenis is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::123456789:role/MyCoolDevRole
```

Yes, even if I fixed the name of that role to be be "DevServiceRole-MyCoolDevRole", I get the same error if I don't also specify the permissions boundary.

```
User: arn:aws:sts::123456789:assumed-role/iam-perm-boundary-DeveloperRole-14W1CA3SPJ6FS/sdevenis is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::123456789:role/DevServiceRole-MyCoolDevRole
```

**In my experience, you won't see errors that explicitly tell you that you missed the permissions boundary**

## Deployment with AWSCLI (On Windows)

Create-Stack
```
aws cloudformation create-stack --stack-name IAM-Permission-Boundary-Example --template-body file://cloudformation/iam-perm-boundary.template --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM --parameters file://cloudformation/parameters.json
```

Update-Stack
```
aws cloudformation update-stack --stack-name IAM-Permission-Boundary-Example --template-body file://cloudformation/iam-perm-boundary.template --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM --parameters file://cloudformation/parameters.json

```

Delete-Stack
```
aws cloudformation delete-stack --stack-name IAM-Permission-Boundary-Example
```