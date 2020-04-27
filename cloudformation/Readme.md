The intent of these cloudformation template and IAM Policy and Role structure.

To create the following roles:

Accenture Provided Admin Role -- is used to create --> Innovation Mine Contributor and R

The complexity with the ContributorROles is that they have permission to create additional IAM Roles which can then be used as part of an application deployment.

IAM is a powerful serivce and it's policy language and capabilities can be difficult to follow.

Its often consudered best practice to add Managed Policies

At a high level, the Contributor roles should be able to:

Create additional roles that don't exceed a certian privilige level.
Be able to create aws service roles.
Be able to pass roles to services.

What they shuould not be able to do is create an Administrator Role.

The cloudformation template really h

Restricting the contributor roles to creating roles under certain paths will cause issues with some services. For example, ECS Using paths 

