# Cloudtrail Service Baselining and Hunting

## Overview
This document will go over concepts on how to baseline your CT log activity. It will also include a triage methodology when investigating events. This guide assumes you are running Splunk but this can be utilized in other log ingestion tooling.


###Cloudtrail API Calls

Most services include the following prefixes for their API calls. These include:

Get
List
Describe
Create
Put
Update
Delete


APIs that make do not make changes to resources or configurations are readOnly. These include Get, List, Describe and etc.
APIs that make changes to resources or configurations are not readOnly. These include Create, Put, Update, Delete and etc.

###Splunk Queries to Baseline

1. Develop a 30 day service baseline of changes within all account[s].

```bash
index=cloudtrail readOnly=False earliest=-30d  | stats count by eventSource | sort -count
```

2. Develop a 30 day baseline for changes within a single Account

```bash
index=cloudtrail readOnly=False aws_account_id=<affectedaccountid> earliest=-30d | stats count by eventSource | sort -count
```

3. Review activity for a given service that involved changes for the past 30 days

```bash
index=cloudtrail readOnly=False aws_account_id=<affectedaccountid> earliest=-30d | stats count by eventName | sort -count
```

4. Users that are making the most changes

```bash
index=cloudtrail readOnly=False earliest=-30d | stats count by userIdentity.principalId | sort -count
```

5. Top API Calls Sources

```bash
index=cloudtrail aws_account_id<accountId> earliest=30d | stats count by SourceIP | sort -count
```

Note: For specific customization Amazon IP ranges are available @ ip-ranges.amazonaws.com/ip-ranges.json

6. Top UserAgents utilized within the past 30 days.

```bash
index=cloudtrail earliest=30d | stats count by userAgent | sort -count
```

```json
User Agent Examples
signin.amazonaws.com – The request was made by an IAM user with the AWS Management Console.
console.amazonaws.com – The request was made by a root user with the AWS Management Console.
lambda.amazonaws.com – The request was made with AWS Lambda.
aws-sdk-java – The request was made with the AWS SDK for Java.
aws-sdk-ruby – The request was made with the AWS SDK for Ruby.
aws-cli/1.3.23 Python/2.7.6 Linux/2.6.18-164.el5 – The request was made with the AWS CLI installed on Linux.
AWS Internal–CloudTrail redacts the value of userAgent and replaces it with AWS Internal if the request was made with a proxy client (such as the AWS Management Console), and sessionCredentialFromConsole is present with a value of true.

Ref: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html
```

7. Principals with the most failed write calls

```bash
index=cloudtrail aws_account_id<accountID> readOnly=False errorCode!=Success earliest=-30d | stats count by userIdentity.principalId eventName errorCode | sort -count
```

8. Principals with the most failed readOnly calls

```bash
index=cloudtrail aws_account_id<accountID> readOnly=False errorCode!=Success earliest=-30d | stats count by userIdentity.principalId eventName errorCode | sort -count
```

8. API calls for unsupported regions

```bash
index=cloudtrail awsRegionearliest=-30d | stats count by awsRegion
```

### Incident Triage in AWS

Here are a few key indicators to help with investigations with initial triage of an AWS incident.


1. Confirm the API calls utilized by the principal for the past 30d.
```bash
index=cloudtrail userIdentity.principalId=<principalId> earliest=-30d | stats count by userIdentity.arn eventName | sort -count

Note: You can seperate these by
```

Confirm scope of services principal utilized within the past 30 days.

```bash
index=cloudtrail userIdentity.principal=<principalId> readOnly=
```
