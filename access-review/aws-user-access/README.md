# AWS Access Review

This workshop series was created to provide guidance on how to perform AWS policy analysis using the JupiterOne Platform. 

The concepts covered in this workshop provide guidance on Access Review related tasks that may be a requirement during a quarterly review or as evidence during an audit. In addition, several of these queries can be used as Insight dashboards and Alert Rules that can be configured to meet the requirements set by your organization's risk and security requirements.

**Prerequisite**
This workshop requires the AWS integration.

In addition, a identity provider (idp) integration (Okta, OneLogin) will provide additional context.

## AWS IAM Policy Review

During an AWS IAM policy review, there are sources of information that will be helpful in discovery. In a IAM policy review, our goal is to discover our current policy objects, which identities or principals are attached to them, and what actions are allowed by those policies on which resources. 

Our first step will be to identify the policies in our organizations environment. As part of any Policy review, the `AccessPolicy` class will provide you with policies across your services. 

**IAM Policies in your production environment:**
`FIND AccessPolicy THAT ALLOWS * WITH tag.Production=true`

This query will show any `AccessPolicy` that has an `ALLOWS` relationship with a entity tagged as production. This may include on Cloud Service Providers (CSPs) `AccessPolicy`. The list below contains the AWS IAM Policy types:

IAM User Policy     `aws_iam_user_policy `
IAM Group Policy    `aws_iam_group_policy`
IAM Role Policy     `aws_iam_role_policy`
IAM Managed Policy  `aws_iam_policy `
//S3 Bucket Policy    `aws_s3_bucket_policy`

If we wanted to target a specific service, we can choose a service and entity type to scope our Access Review. In the query below, we will evaluate the AWS S3 Service along with AWS S3 Buckets type:

**AWS S3 Buckets and Service with Access Policies that ALLOW actions:**
`Find (aws_s3_bucket|aws_s3) THAT ALLOWS AccessPolicy RETURN TREE`
//Screenshot

In our example above, we use `RETURN TREE` to show the graph view in the JupiterOne platform. In addition, we used the `(|)` operators to select multiple types. In the query above, we will see that not only can an AccessPolicy ALLOWS access to an individual bucket, but as well as the S3 Service.

If you click the ALLOWS relationship, you will see that the actions associated with the policy attached to the resource. 
//Screenshot

We will want to generate a report that can show these details in a spreadsheet or JSON format. We can use aliasing to just that:

**S3 Policy Review - Actions and Resources**
`Find (aws_s3_bucket|aws_s3) that ALLOWS as rule AccessPolicy as policy Return policy.displayName, rule.actions, rule.resources`

In the example above, we as `as rule` and `as policy` to alias our `ALLOWS` relationship and entity. We can then use `RETURN` to choose which properties we want to include.

We may find that certain actions are over-privileged. In that case, we can filter for actions that are of unnecessary or dangerous:
**Policies that have `:*` configured as an action:**
`Find (aws_s3_bucket|aws_s3) that ALLOWS as rule AccessPolicy as policy WHERE rule.actions~=":*" Return policy.displayName, rule.actions, rule.resources`

This can be turned into an Alert using the alert rules processes: (Link to support docs)

We've identified that actions and resources attached and configured in our AWS environment. Now lets evaluate the principle. A principle can be a role, user group, or user. Lets see what principle are directly assigned to a policy:
**S3 Principle:**
`Find (aws_s3_bucket|aws_s3) that ALLOWS as rule AccessPolicy as policy THAT ASSIGNED (AccessRole|UserGroup|User) as principles Return Tree`

We should notice that a policy can be `ASSIGNED` to a `User`, `AccessRole`, and `UserGroup`. For our next query, we can use the optional traversal operators to include the scenarios where users can be assigned access through a `AccessRole`, or `UserGroup`:

**S3 Assigned Roles, UserGroups, and Users:**
`Find (aws_s3_bucket|aws_s3) that ALLOWS as rule AccessPolicy as policy (THAT ASSIGNED AccessRole)? (THAT ASSIGNED UserGroup)? THAT (ASSIGNED|HAS) User Return Tree`

Lastly, lets identify any User that can access our AWS environment:
**All Users who have access to AWS Environment:** 
`Find UNIQUE User (THAT (ASSIGNED|HAS) UserGroup)? (THAT ASSIGNED AccessRole)? THAT ASSIGNED AccessPolicy THAT ALLOWS * with _type^="aws"`
