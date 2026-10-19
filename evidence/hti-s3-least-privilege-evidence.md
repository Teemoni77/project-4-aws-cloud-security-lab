# HTI Cloud Security Lab --- S3 Least-Privilege Policy Evidence

## Objective

Validate that a customer-managed IAM policy grants read-only access to
the designated HTI cloud security lab S3 bucket, without allowing
uploads or deletions.

## Policy

**Policy name:** `HTICloudLabS3ReadOnly`\
**Policy type:** Customer managed\
**Version:** `2012-10-17`\
**Scope:** One lab bucket and its objects

The account-specific bucket ARN is intentionally represented by a
placeholder. Do not publish account IDs or sensitive account details.

``` json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListSpecificLabBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::[REDACTED-LAB-BUCKET]"
    },
    {
      "Sid": "ReadObjectsInSpecificLabBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::[REDACTED-LAB-BUCKET]/*"
    }
  ]
}
```

## Validation results

The policy was attached temporarily to a test IAM user and tested using
AWS CLI. The test identity and credentials were removed after testing.

  -----------------------------------------------------------------------
  Test                    Expected outcome        Observed outcome
  ----------------------- ----------------------- -----------------------
  List objects in the lab Allowed                 **Allowed** --- command
  bucket                                          returned the lab object
  (`s3:ListBucket`)                               listing

  Read the test object    Allowed                 **Allowed** --- command
  (`s3:GetObject`)                                returned the object
                                                  contents

  Upload a test object    Denied                  **Denied** --- AWS
  (`s3:PutObject`)                                returned
                                                  `AccessDenied`; no
                                                  identity-based policy
                                                  allowed the action

  Delete a test key       Denied                  **Denied** --- AWS
  (`s3:DeleteObject`)                             returned `AccessDenied`
  -----------------------------------------------------------------------

The delete test targeted a test key that did not exist; it did not
delete a real lab object.

## Security interpretation

-   The policy grants only `s3:ListBucket` on the specified bucket and
    `s3:GetObject` on objects within it.
-   It does not grant `s3:PutObject` or `s3:DeleteObject`.
-   The observed CLI results confirmed the intended allow/deny behavior
    for the tested actions.
-   This is evidence for the tested S3 permissions only. It does not
    prove that every other AWS permission or account-level control is
    correctly configured.

## Evidence artifacts

-   `hti-s3-readonly-policy-redacted.png` --- redacted AWS IAM policy
    screenshot.
-   `hti-s3-putobject-denied-redacted.png` --- redacted CLI evidence for
    the denied upload test.

## Publication hygiene

Before publishing, ensure screenshots, command output, README files, and
repository history do not expose AWS account IDs, access keys, session
tokens, private IP addresses, or other account-specific details. Never
commit AWS credentials.
