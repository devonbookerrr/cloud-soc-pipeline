# Cloud SOC Pipeline

As a SOC analyst at an MDR, I investigate restricted country logins, IAM policy changes, suspicious config modifications daily. And it's been great because it has given me the opportunity to talk to our customers about why the alert alerted. But the reality is, not many smaller companies have the money or resources to invest into a human-monitored 24x7 SOC.

This project aims to be my way of helping smaller companies see the benefit of having cloud security visibility, even if they don't have the "money" for it.

## What the project entails

CloudTrail captures all API activity to S3. EventBridge rules fire on suspicious events you already know from your boards - console logins from flagged geolocations, IAM changes, security group modifications. A Python Lambda function picks up the event, queries historical context from RDS, then sends the alert payload to AWS Bedrock (Claude) which acts like a junior analyst - it reads the event, checks it against your runbook logic, scores severity, and writes a plain-English summary. Results log to RDS. SNS notifies the team. You query trends with SQL.

## Tools hit

Lambda, S3, RDS, IAM, EventBridge, SNS, Bedrock/GenAI, Python, SQL, Linux, Git

