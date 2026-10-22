# IAM Account Access and Console Session Review

## 1. Objective

Review the account-access role and selected AWS CloudTrail console sign-in records as part of the Project 4 cloud security lab. The purpose is to document the observed configuration, assess what the available evidence supports, and identify sensible follow-up actions without changing AWS-managed access configuration.

## 2. Scope and evidence reviewed

The review used the AWS Console screenshots and CloudTrail event records inspected during this lab session, including:

- The `AccountFullAccessRole` summary, permissions, and trust relationship.
- The AWS-managed `AdministratorAccess` policy attached to the role.
- Selected `ConsoleLogin` events associated with the Builder ID sign-in flow.
- The configured authenticator-app MFA method and the observed fresh sign-in behavior.

This is a focused review of the evidence available during the session, not a complete account compromise assessment or a review of every sign-in event.

## 3. Observations

### 3.1 Account access role

The account uses an AWS-managed role named `AccountFullAccessRole`. The role's displayed permissions include the AWS-managed `AdministratorAccess` policy, which grants broad access to AWS services and resources. The role trust relationship shown in the console allows the AWS service principal `account-access.amazonaws.com` to call `sts:AssumeRole` and `sts:SetContext`.

This configuration is consistent with the observed AWS-managed, Builder ID-mediated account access flow. The role is not a normal IAM user, and the lab should not attempt to delete it or casually change its permissions or trust policy.

### 3.2 Console sign-in records

The reviewed CloudTrail `ConsoleLogin` records showed:

- Successful console sign-ins using the Builder ID flow.
- An assumed-role identity associated with `AccountFullAccessRole`.
- The `MFAUsed` field recorded as `No` in the displayed console-login events.
- The session context showed `mfaAuthenticated: false`.

These fields describe the recorded session and event context. On their own, they do not prove that an attacker signed in or that the configured authenticator method was bypassed. During the investigation, the user reported that an authenticator app was listed as an active MFA method and that a fresh sign-in after fully signing out prompted for MFA. Existing Builder ID or browser sessions may explain why some returns to the console did not prompt again, but that explanation has not been independently established from these records.

The user recognized the reviewed sign-in times as falling within a period when they were active. This is useful context, but it is not independent proof of who initiated each event.

### 3.3 Region and event interpretation

The console-login events were viewed in CloudTrail Event history in the region displayed for each event. CloudTrail event history is regional, so conclusions from a single region's event list should not be treated as proof that no other relevant events exist elsewhere.

### 3.4 Resource usage

At the time of this review, the AWS billing and Cost Explorer views showed no recorded cost and the displayed promotional credit balance remained intact. This is reassuring context for the lab, but it does not by itself rule out unauthorized access: an unauthorized session can inspect resources or perform actions that incur no immediate charge. Resource usage and sign-in evidence must be assessed separately.

## 4. Risk assessment

**Assessment: Inconclusive; no confirmed compromise identified in the evidence reviewed.**

The broad permissions on `AccountFullAccessRole` make it important to protect the Builder ID sign-in and session. However, the role's name, AWS-managed trust relationship, and observed Builder ID sign-in pattern are not, by themselves, indicators of compromise.

The `MFAUsed: No` and `mfaAuthenticated: false` fields deserve documentation and continued verification, but they should not be described as proof of MFA failure given the observed Builder ID flow and the successful MFA prompt during a fresh sign-in.

No conclusion is drawn that the account was compromised or that it was definitively uncompromised.

## 5. Recommended follow-up

1. Keep the authenticator-app MFA method enabled and confirm that a genuinely fresh authentication requires the expected MFA challenge.
2. Continue reviewing successful and failed `ConsoleLogin` events across relevant regions, comparing timestamps and source addresses with the user's known activity. An IP address is contextual evidence, not proof of identity.
3. Review CloudTrail for unexpected IAM, STS, S3, networking, compute, and billing-related management events. Investigate any event that cannot be explained by known lab activity.
4. Continue monitoring billing and resource inventory. Do not use unchanged credits or zero cost as the sole basis for deciding whether access was legitimate.
5. Do not modify or delete the AWS-managed `AccountFullAccessRole` as part of this exercise. If AWS Support clarifies the managed-role permission model, document that response before considering any access changes.
6. Keep screenshots and exported event evidence sanitized before publishing. Do not commit session tokens, access keys, MFA secrets, private account details, or unnecessary source IP addresses.

## 6. Limitations

- The review covered selected console events and screenshots, not an exhaustive export of all relevant CloudTrail events.
- The available evidence does not establish the exact Builder ID/browser session behavior that led to some sign-ins occurring without a new MFA prompt.
- The review did not establish that all data events or all regions were covered.
- Billing observations are time-specific and may lag behind activity.
- No changes were made to the AWS-managed role or its policies as part of this review.

## 7. Outcome

**Status: Documented — follow-up monitoring recommended.**

This review demonstrates practical experience with:

1. Inspecting AWS IAM role permissions and trust relationships.
2. Interpreting CloudTrail console-login and assumed-role context.
3. Distinguishing a recorded MFA/session attribute from proof of compromise.
4. Correlating identity evidence with billing and resource observations without overclaiming.
5. Documenting findings, limitations, and safe follow-up actions.

## Author

**Taiwo Ibrahim**

Cybersecurity | Cloud Security | Security Operations
