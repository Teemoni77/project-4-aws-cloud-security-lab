# Project 4: AWS Cloud Security Lab

## Overview

This project documents hands-on cloud security exercises using Amazon Web Services (AWS), with an emphasis on identity and access management, storage security, least privilege, and audit evidence.

The lab is being developed incrementally. This repository records implemented configurations, observed test results, and investigation notes.

## Objectives

- Apply least-privilege access using AWS IAM policies.
- Secure S3 storage with public-access blocking, encryption, and versioning.
- Validate allowed and denied actions through practical testing.
- Review CloudTrail management events to understand AWS audit evidence.
- Build practical cloud security skills relevant to defensive security operations.

## Lab Environment

| Component | Configuration |
|---|---|
| Cloud provider | Amazon Web Services (AWS) |
| Storage | Amazon S3 |
| Identity and access management | AWS IAM |
| Audit logging | AWS CloudTrail Event history |
| S3 region | Europe (Stockholm), `eu-north-1` |
| CloudTrail region reviewed | US East (N. Virginia), `us-east-1` |

## Completed Exercise: IAM Least-Privilege Testing

A customer-managed IAM policy named `HTICloudLabS3ReadOnly` was created to permit listing and reading objects in a specific lab S3 bucket.

The policy was attached to a temporary IAM test user, and actual AWS CLI operations were used to validate its permissions.

| Test | Result |
|---|---|
| List objects in the lab bucket | Allowed |
| Read a test object | Allowed |
| Upload an object | Denied |
| Delete an object | Denied |

The temporary test access key and IAM user were deleted after testing. CloudTrail management events were inspected to verify the IAM user creation, policy attachment, access-key deletion, and user deletion.

These are separate evidence sources: the CLI results establish the observed S3 permission behavior, while the reviewed CloudTrail records establish the corresponding IAM management activity.

**Detailed report:** [IAM least-privilege policy testing and CloudTrail audit](evidence/iam-least-privilege-policy.md)

## Account Access Review

A focused review documented the AWS-managed `AccountFullAccessRole` configuration and selected Builder ID-associated `ConsoleLogin` records. The observed event fields include session-level MFA indicators that require careful interpretation; the reviewed evidence does not confirm compromise or definitively rule it out. The report records evidence limitations and safe follow-up checks without changing the AWS-managed role.

**Detailed report:** [IAM account access review](evidence/iam-account-access-review.md)

## Security Controls Observed

- S3 Block Public Access enabled.
- S3 Object Ownership set to Bucket owner enforced, with ACLs disabled.
- Default S3 server-side encryption using SSE-S3.
- S3 bucket versioning enabled.
- IAM permissions restricted to the intended lab bucket and its objects.

## Repository Structure

- `README.md` — project overview, scope, and progress.
- `evidence/README.md` — index of evidence and publication guidance.
- `evidence/iam-least-privilege-policy.md` — redacted policy configuration, test results, CloudTrail observations, and cleanup notes.
- `evidence/iam-account-access-review.md` — account-access role and selected console sign-in review, including limitations and follow-up actions.
- `evidence/` — selected screenshots or sanitized evidence, where appropriate.

## Scope and Limitations

This is a developing lab, not a production security assessment. The completed exercise validates the listed S3 actions under the tested IAM identity and policy configuration. It does not claim comprehensive testing of every possible access path or complete coverage of S3 data events in CloudTrail. The account-access review is limited to selected events and configuration screenshots; it is not a full incident-response investigation.

Further exercises will expand the lab to cloud logging, monitoring, identity security, and detection and investigation workflows.

## Author

Taiwo Ibrahim

Cybersecurity | Cloud Security | Security Operations
