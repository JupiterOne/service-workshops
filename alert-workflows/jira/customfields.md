If you're configuing custom fields into your instance of the Jira integration, or plan to use them in tickets, an important first step is determining the field names.

Jira custom fields are just labels or fields attached on Jira tickets that provide process logic for use across the business. 
https://confluence.atlassian.com/adminjiraserver/managing-custom-fields-1047552711.html

If you plan to use the J1 Jira integration, you will need to provide a list of your relevant custom fields within its respective configuration. This will allow those fields to appear in their respective `jira_issue` entities as properties. You may also use those fields within J1 Alert actions that generate Jira tickets. Jira custom fields do not retain their field name when queried over the API nor do they respect their names when tickets are published. Instead they follow a naming convention as follow: `customfield_10123`

You can follow this guide to obtain a list of your custom fields: https://confluence.atlassian.com/jirakb/how-to-find-any-custom-field-s-ids-744522503.html

### Recommendations and Best Practices

It is highly recommended that you only use fields from projects you plan to ingest `jira_issue` from, as well as fields that are releveant to your ticket creation procress. There are likely many fields that are not relevant to your JupiterOne use-case.
One way to obtains the fields that will be relevant to your use-case is to select a jira issue that contains the relevant fields, and enter the following URL into your browser. Please note, you will need to replace `<JIRA_BASE_URL>` and `<JIRA-ISSUE>` with their respective values.
`<JIRA_BASE_URL>/rest/api/latest/issue/<JIRA-ISSUE>?expand=names` 

Example:
https://jupiterone.atlassian.net/rest/api/latest/issue/TRIAGE-112?expand=names
```
{
...
  "key": "TRIAGE-123",
  "names": {
    "customfield_10111": "Vendor Source",
    "customfield_10112": "SLA",
    "labels": "Labels",
    "status": "Status",
...
```

Note that fields like `label`, `status`, and many others are standard fields. These fields will not need to be included in your JupiterOne integration configration, but can be referenced in your Alert logic.
