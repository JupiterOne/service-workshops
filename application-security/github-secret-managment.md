# GitHub Secret Management
This Workshop will provide guidance on how to user JupiterOne in auditing GitHub secrets.

This concept is relevant to GRC teams who wishes to audit current GitHub Secret best practices. They are also relevant to analysts or engineers who need to understand the blast radius of a secret's exposure, as well as alert on improper uses of GitHub secrets.

The information obtained in this query can be used to provide evidence for "High Level Application Security Requirements" procedure and control(cp-sdlc-appsec-req), as well as be customized for alerting and insight dashboards.

**Prerequisite**

This workshop requires the GitHub integration.

The session will use concepts shown in the "Intro to J1QL" workshop.

The concepts covered in this session reference the following blog post: https://try.jupiterone.com/blog/github-secrets-management-with-jupiterone

## GitHub Secrets

JupiterOne is able to ingest metadata relating to your Encrypted GitHub secrets. GitHub Secrets are useful for storing sensitive data relating to your org, repos, environment, or other aspects to your softwares business logic.
JupiterOnes observability into GitHub Secrets allows you to audit their availability, determine their last update (rotation schedule), the total securities footprint, and any problems associated with that secret. 

### Current GitHub Secrets

JupiterOne will relate a GitHub secret to the code repository that contains it. That relationship can be shown with the following query:

```
Find CodeRepo 
    that (USES|HAS) Secret 
    return CodeRepo.displayName, 
    Secret.displayName ORDER BY Secret.displayName ASC

```
The query above will show current `CodeRepo` that leverage a `Secret`.
> The `Secret` class can include other secrets. The types for GitHub secrets include the follow:
> - `github_env_secret`
> - `github_repo_secret`
> - `github_org_secret`

### Secret Rotation

`Secret` entities have a `createdOn` and `updatedOn`. These properties can be used to audit the rotation of secrets:

```
Find Secret with updatedOn < date.now - 180 days
```
The query above shows GitHub Secrets that have not been updated within the last 180 days.

> The `date.now` function along with the date can be set to specific dates. It is also possible to select larger time increments that days. Please reference the following documentation on our date function:

### Secret Override

GitHub's implementation of secrets allows there to be different tiers of secrets: repo, environment and org. It cases where rotation is necessary, it should be worth nothing that secrets may override one another making rotation at multiple levels necessary:

```
Find CodeRepo 
    that USES Secret 
    that OVERRIDES >> Secret RETURN TREE
```

The query above will show any overridden secrets. In some cases this may be intentional, but is necessary to audit upon rotation.

>If a `CodeRepo` `USES` a `github_repo_secret` or `github_env_secret`, and a `github_org_secret` of the same name exists, JupiterOne will create an `OVERRIDES` relationship.

### Secrets with Problems

If the queries above have been used to create alerts, JupiterOne can be used to identify secrets with a `Problem`:
```
Find Secret 
    that HAS Problem
```
The query above would surface any `Secret` that has an `Alert` associated with it of non-informational severity, or that has a compliance gap.

>`Problem` is a JupiterOne class that can be used to identify entities with any Alert of compliance gap associated with them. From the JupiterOne graph view, `Problem` entities will have a red dot on their node.
