# devtech-iam

This repo demonstrates how to use AWS IAM Permisisons Boundaries as a mechanism for allowing dev teams to create their own IAM roles while maintaining control of the scope of permissions on those roles.

## Overview

This template creates 3 resources:

|Resource|Description|
|---|---|
|DeveloperRole|The IAM role that developers work under. 
|DeveloperRolePolicy|The IAM policy attached to the above role. This policy has conditional permissions to create additional IAM resources.
|PermissionBoundaryPolicyForRolesCreatedByDeveloper| The IAM policy that defines the permissions boundary for roles created by the DeveloperRole.

A Developer can create an IAM role as long as they satisfy two conditions:

1) The role name must begin with Developer-
2) They must reference the permissions boundary policy created in this template.

Additonally, if they are creating an IAM policy:

3) The policy name must being with Developer-


## Notes and FAQ

### How does this actually restrict access?

Its probably best to read the AWS doco in full, but in short: A dev can only create and delete roles and policies that match the regex in the template (currently Developer-*). From a policy perspective, although they can create (or select an existing) policy with unlimited scope, they cannot attach that policy to a role without supplying the name of the permission boundary policy. The permission boundary policy then squashes the effective permissions.

### Be carful with assigning more IAM permissions

In IAM, an explicit deny always trumps any allow. So you might think its ok to assign the dev role AdministratorAccess or iam:* with the thought that the more granual IAM permissions will override the *. This is not the case, as this policy defines conditional allows. If you give the dev role more permissions than this is what they will get as the effective permissions.

### Why have you split up permissions like CreateRole and DeleteRole between SIDs?

This one's a bit tricky. Essentially, the API's that create and update things accept both the Regex conditional in the ARN (Developer-*) and a PermissionsBoundary attribute. The delete and list method accept the Regex conditional in the ARN but DON'T accept permissions boudaries.

So for something like "iam:DeleteRole" if you list it in the SID with iam:CreateRole (which requires both), you end up in a situation where IAM internal logic expect you to satisfy both rquirements but the IAM API won't accept the permission boundary.

### What do errors look like?

If the dev role tries to create a role that doesn't match the regex (Developer-) OR doesn't supply the name of the Permissions Boundary role, they get the same error, example below.

```
User: arn:aws:sts::123456789:assumed-role/IAM-Dev-Role-DeveloperRole-14W1CA3SPJ6FS/sdevenis is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::123456789:role/MyCoolDevRole
```

Yes, even if I fixed the name of that role to be be "Developer-MyCoolDevRole", I get:

```
User: arn:aws:sts::123456789:assumed-role/IAM-Dev-Role-DeveloperRole-14W1CA3SPJ6FS/sdevenis is not authorized to perform: iam:CreateRole on resource: arn:aws:iam::123456789:role/Developer-MyCoolDevRole
```

**In my experience, you won't see errors that explicitly tell you that you missed the permissions boundary**

## Deployment with AWSCLI

Create-Stack
```
aws cloudformation create-stack --stack-name IAM-Dev-Role --template-body file://cloudformation/iam-dev-role.template --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM
```

Update-Stack
```
aws cloudformation update-stack --stack-name IAM-Dev-Role --template-body file://cloudformation/iam-dev-role.template --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM CAPABILITY_IAM
```