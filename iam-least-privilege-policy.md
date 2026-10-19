# IAM Least-Privilege Policy Testing and CloudTrail Audit

## 1. Objective

Validate an AWS IAM policy that permits read-only access to a specific Amazon S3 bucket while denying object uploads and deletions. Confirm the observed permissions using AWS CLI and review CloudTrail management events to document the temporary test user's lifecycle and cleanup.

## 2. Lab Environment

| Component                  | Details                            |
| -------------------------- | ---------------------------------- |
| Cloud provider             | Amazon Web Services (AWS)          |
| Services                   | IAM, Amazon S3, AWS CloudTrail     |
| S3 region                  | Europe (Stockholm), `eu-north-1`   |
| CloudTrail region reviewed | US East (N. Virginia), `us-east-1` |
| IAM policy                 | `HTICloudLabS3ReadOnly`            |
| Temporary test user        | `hti-s3-policy-test`               |
| Test object                | `s3-encryption-test.txt`           |

## 3. Least-Privilege Policy

A customer-managed IAM policy named `HTICloudLabS3ReadOnly` was created and attached to a temporary test user.

The policy grants only these two actions:

- `s3:ListBucket` — list objects in the designated lab bucket.
- `s3:GetObject` — read objects within that bucket.

It does not grant `s3:PutObject` or `s3:DeleteObject`.

### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListSpecificLabBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::hti-cloud-security-lab-875691018505-eu-north-1-an"
    },
    {
      "Sid": "ReadObjectsInSpecificLabBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::hti-cloud-security-lab-875691018505-eu-north-1-an/*"
    }
  ]
}
```

**Policy ARN:** `arn:aws:iam::875691018505:policy/HTICloudLabS3ReadOnly`

The policy is scoped to the specific lab bucket rather than granting read access to all S3 buckets.

## 4. Practical Permission Tests

The policy was tested using AWS CLI with a dedicated profile for the temporary IAM user.

| Operation                      | Expected outcome | Observed result                        |
| ------------------------------ | ---------------- | -------------------------------------- |
| List objects in the lab bucket | Allowed          | **PASS** — listing succeeded           |
| Read `s3-encryption-test.txt`  | Allowed          | **PASS** — object contents returned    |
| Upload `hti-deny-test.txt`     | Denied           | **PASS** — AWS returned `AccessDenied` |
| Delete `hti-deny-test.txt`     | Denied           | **PASS** — AWS returned `AccessDenied` |

The read operation returned the contents of the existing test object:

`HTI Cloud Security Lab - Version 2 test.`

The attempted upload was denied, so the upload test did not create the target object. The deletion attempt targeted that same test key and was also denied.

### Test conclusion

All four observed outcomes matched the intended policy behavior. The test demonstrated that the user could list and read objects in the designated bucket but could not upload or delete objects using the tested permissions.

These results validate the tested operations under this identity and policy configuration; they do not establish that every possible S3 access path was tested.

## 5. CloudTrail Audit Evidence

CloudTrail Event history was reviewed in `us-east-1`. The following management events were found and inspected.

| Event              | Timestamp (Nigeria time, UTC+01:00) | Purpose                                      |
| ------------------ | ----------------------------------- | -------------------------------------------- |
| `CreateUser`       | 19 October 2026, 18:25:02           | Records creation of the temporary IAM user   |
| `AttachUserPolicy` | 19 October 2026, 18:25:03           | Records attachment of the read-only policy   |
| `CreateAccessKey`  | 19 October 2026, 19:01:05           | Records creation of the temporary access key |
| `DeleteAccessKey`  | 19 October 2026, 19:39:54           | Records deletion of the temporary access key |
| `DeleteUser`       | 19 October 2026, 19:41:33           | Records deletion of the temporary IAM user   |

The inspected event records showed no error code. Their request and response details identified the relevant user, policy, or access-key operation.

**Important distinction:** The CloudTrail events above document IAM management activity. The S3 allow/deny results were observed through AWS CLI. We did not establish that the S3 object-level requests were recorded in CloudTrail, because S3 data events were not enabled for this test.

## 6. Cleanup

After testing:

- The temporary access key was deleted.
- The temporary IAM user `hti-s3-policy-test` was deleted.
- The customer-managed policy `HTICloudLabS3ReadOnly` remains available in the account.
- The lab bucket and its test object remain separate lab resources.

The temporary CLI profile `hti-policy-test` may still exist in the CloudShell configuration, but its access key has been deleted. It should not be reused; remove the stale profile configuration when convenient.

## 7. Security Observations

- **Least privilege:** Permissions were restricted to listing and reading within one designated bucket.
- **Permission validation:** Both permitted and denied actions were tested directly rather than relying only on a policy simulator.
- **Auditability:** CloudTrail management events provided evidence of IAM setup and cleanup.
- **Credential hygiene:** The temporary key and user were removed after testing.
- **Evidence limitations:** The tests cover the four listed operations and the inspected management events, not a comprehensive assessment of the AWS account.

The source IP recorded in CloudTrail matched the public IPv4 address observed during a subsequent check on the same internet connection. This is supporting network evidence, not proof of an individual's identity.

The inspected assumed-role session showed `mfaAuthenticated: false`. This is a session-level observation and does not establish whether MFA is configured for every account or sign-in mechanism.

## 8. Outcome

**Status: PASS — the tested least-privilege policy behaved as intended.**

This exercise demonstrates practical experience with:

1. Writing a resource-scoped IAM policy.
2. Applying least-privilege permissions to Amazon S3.
3. Testing allowed and denied operations using AWS CLI.
4. Interpreting AWS `AccessDenied` responses.
5. Reviewing CloudTrail management events.
6. Removing temporary IAM credentials and test identities after validation.

## Author

**Taiwo Ibrahim**

Cybersecurity | Cloud Security | Security Operations
