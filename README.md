# Cloud SOC Pipeline

I’m a SOC analyst at an MDR, and I investigate the same categories of alerts every day: restricted-country logins, IAM permission changes, and suspicious configuration modifications. I’ve gotten a lot of value out of that work because it forces me to understand what happened, why it mattered, and how to explain it clearly to customers.

But the reality is that most smaller companies don’t have the budget (or staffing) for a 24x7 human-monitored SOC.

This project is my attempt to bridge that gap by building a cloud-native detection pipeline that gives smaller teams meaningful security visibility in AWS — and pairs those detections with clear, analyst-style context so the alert is actually actionable.

## What the project entails

CloudTrail captures all API activity to S3. EventBridge rules fire on suspicious events you already know from your boards - console logins from flagged geolocations, IAM changes, security group modifications. A Python Lambda function picks up the event, queries historical context from RDS, then sends the alert payload to AWS Bedrock (Claude) which acts like a junior analyst - it reads the event, checks it against your runbook logic, scores severity, and writes a plain-English summary. Results log to RDS. SNS notifies the team. You query trends with SQL.

## Tools hit

Lambda, S3, RDS, IAM, EventBridge, SNS, Bedrock/GenAI, Python, SQL, Linux, Git

### Week 1 - Environment setup (getting to “ready to build”)

Before I wrote any detection logic, I treated this like a real build: secure the account, set up a clean dev workflow, and make sure I can actually ship code.

Here’s what I did in Week 1:

- Created and secured my AWS account (MFA enabled immediately, no root usage).
- Created an admin IAM user for initial setup (with the intent to tighten permissions later as the architecture solidifies).
- Set a small billing alert so I can safely experiment.
- Built my local environment on Windows using WSL2 + VS Code (AWS CLI, Python, pip, Git).
- Verified CLI access with `aws sts get-caller-identity`.
- Created the GitHub repo and wrote the first version of this README so the “why” of the project is clear (not just the tech).
- Started a `/samples` folder so I can keep real CloudTrail examples handy while building detections.

The goal of Week 1 was simple: remove friction. If I can’t reliably test, commit, and iterate, the pipeline never gets finished.

---

### Week 2 - Getting logs flowing (and generating real security-relevant activity)

Week 2 was about proving the foundation: CloudTrail is logging correctly, logs land in S3, and I can trigger the kinds of events a real SOC cares about.

What I set up:

- Enabled CloudTrail management event logging.
- Configured CloudTrail to deliver logs to an S3 bucket.
- Waited for logs to populate, downloaded them, and reviewed them manually in VS Code to understand the raw event structure (what AWS actually records vs. what we usually see after normalization in a SOC tool).

Then I generated test activity on purpose (a mini “red team against my own account”), so I’d have real data to build detections from.

#### The real events I generated in Week 2 (CloudTrail samples)

These are the types of events I triggered and captured in CloudTrail (from my own account activity):

1) ConsoleLogin (successful login; iPhone user-agent, MFA used)  

2) CreateUser (created an IAM user: `iamuser`)  

3) AttachUserPolicy (attached `AdministratorAccess` to `iamuser`)  

4) AuthorizeSecurityGroupIngress (opened SSH `22` to `0.0.0.0/0` on a security group)  

5) PutBucketPolicy (CloudTrail bucket policy write for my project log bucket: `soc-pipeline-cloudtrail-log-db`)  

6) PutBucketPolicy (CloudTrail bucket policy write for the auto-named CloudTrail log bucket)  

7) DeleteUser (deleted the IAM user: `iamuser`)

The point of including these isn’t “look, I clicked around in AWS.” It’s to show that this pipeline is being built against real CloudTrail output, using the same categories of activity that drive real-world detection and response: authentication events, identity changes, network exposure, and storage policy changes.

#### Week 2 deliverable

By the end of Week 2, I had:

- CloudTrail actively logging
- S3 receiving logs
- Multiple real security-relevant events captured and saved for reference
- A clearer understanding of the fields the pipeline will rely on (e.g., `eventName`, `eventTime`, `sourceIPAddress`, `userIdentity`, region, and request parameters)
