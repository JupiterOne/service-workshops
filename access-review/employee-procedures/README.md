# User and Employee Management

This workshop series was created to provide guidance on how to perform User account management and access review.

The concepts covered in this workshop provide guidance on Access Review related tasks that may be a requirement during a quarterly review or as evidence during an audit. These concepts are also highly relevant to a {} who is asked to perform an employee termination or onboarding. 

In addition to the data obtained through this session, several of these queries can be used as Insight dashboards and Alert Rules that can be configured to meet the requirements set by your organization's risk and security requirements.

**Prerequisite**

This session benefits from an identity provider (idp) integration (Okta, OneLogin) being currently configured into the platform. Be ware that additional details will depending on the types of user accounts currently ingested into the platform. For all integration specific related relationships and entities, it is recommended to read the AskJ1 Documentation for the Vendor in question.

## User Account Review

The `User` class contains any user type ingested through the platform. The query below will list all user accounts ingested into JupiterOne
```
FIND User
```

A helpful first step to evaluate the types of user account's in JupiterOne is the check the **Assets** app, and select _User_. 
<Screenshot>

The types of user's available will be dependant on the information currently ingested. In the **Assets** app view, the list of user's can be selected to provide you a list of the subset.

>The typical scheme used in JupiterOne for user account type's is `vendor_user`. Some common examples include:
> - `okta_user`
> - `google_user`
> - `aws_iam_user`
> - `onelogin_user`

It is also worth noting that each user type will include properties relevant to the user vendor in question. These can include email, username, creation date, and many other. Each integration managed user will also have relationships with other entities relevant to Vendor. 

```
FIND okta_user WITH status="ACTIVE"
    THAT HAS okta_user_group with displayName~="Admin"
```
This query would return a list of active Okta User's who are in a group containing the name Admin.

## User Account Status

User account activity can be based on a few properties. These properties can be used to assess if an employee's account is active within their respective Service.  Service's can denote the status of a user in a number of different ways.

### Terminated User Accounts
The example queries below show deactivated user's from their respective accounts.

_All deprovisioned Okta user accounts:_
```
find okta_user with status="DEPROVISIONED"
```
_All Google user with archived accounts:_
```
find google_user with archived=true
```
_All deactivated Bamboo HR user accounts:_
``` 
find bamboohr_user with active=false
```
_All deactivated OneLogin user accounts:_
```
FIND onelogin_user with status=0
```
<TBD - azure_user with accountEnabled=true>

>The property `active=false` may be used in some cases to highlight terminated users, but can also be used for other status', such as suspended or locked accounts.
    
### Inactive User
    
```
find okta_user with active=true and lastLoginOn < date.now-90 day
```

## Relating User Accounts Across Different Services

While two user accounts from different services owned by the same user can not be directly related, it is possible to relate them through the `Person` entity

```
FIND Person
    THAT IS User
    RETURN TREE
```
The example above will show all user account associated with each Person in your JupiterOne account.

>The `Person` entity is created by JupiterOne when your IdP's user accounts are ingested. It is used to correlate related user accounts across the same entity. The relationships that are created to show correlation between accounts is done through our global mapping rules: https://community.askj1.com/kb/articles/796-global-mappings


### Offboarding Review

_The example shows a list of terminated OneLogin user's with active accounts in other services_:
```
FIND onelogin_user with status=0
    THAT IS Person
    THAT IS User with active=true
```


### Employee Onboarding

### User Endpoint management
(this may be a different workshop)

### Employee Training 

### Policy Review


### User Endpoint management
(this may be a different workshop)
