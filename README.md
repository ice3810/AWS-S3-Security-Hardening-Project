## 🛡️ AWS Cloud Security Project: S3 Defense-in-Depth Verification

This project documents the implementation and successful validation of a robust security model on AWS S3, a core exercise demonstrating **Least Privilege**, **Data Encryption**, and **Separation of Duties** principles.

-----

## 1\. 🎯 Use Case & Project Goal

### Primary Use Case

The architecture simulates securing a repository for **sensitive corporate data** (e.g., financial records or proprietary customer information) in the cloud bucket: `my-ice-bucket-1`.

The main objective was to eliminate the **single point of failure** commonly found in cloud storage: the ability for a data handler to also perform administrative or malicious actions.

### Security Goals

1.  **Least Privilege:** Ensure the data handler (`test-readonly-user`) can **only read** the files and nothing else.
2.  **Data Protection:** Guarantee data is secure both **at rest** (encryption) and **in transit** (HTTPS).
3.  **Separation of Duties:** **Isolate** the audit log (`my-logs-ice-1`) from the user, protecting the forensic evidence.

-----

## 2\. ⚙️ The Security Architecture: Full Working Explanation

The solution uses a three-layered security strategy built on the highest authority in access control: the **Explicit DENY** statement.

### Layer 1: Identity Control (The "Allow" Rule)

  * **Component:** **IAM Identity Policy** (`least-privilege-s3-readonly.json`).
  * **Function:** This policy granted the user **ALLOW** permission for the minimum necessary actions: viewing the file list (`s3:ListBucket`) and downloading files (`s3:GetObject`).

### Layer 2: Resource Guardrails (The Mandatory DENY Rules)

  * **Component:** **S3 Resource Policy** (`final-secure-bucket-policy.json`).
  * **Function:** This policy was attached directly to the data bucket to enforce two non-negotiable rules for everyone:
      * **Denial of Write/Delete:** Explicitly **DENIES** all put/delete actions (`s3:PutObject`, etc.), ensuring data integrity is never compromised by any user.
      * **Denial of Insecure Transport:** Explicitly **DENIES** all traffic that is not encrypted, enforcing **HTTPS-only** access.

### Layer 3: Configuration Baseline

  * **Controls:** **Block Public Access** was globally enabled, and **SSE-KMS Encryption** was enforced as the default for all data stored in the bucket.

-----

## 3\. ✅ Validation: Proof of Successful Denial

Security was validated by attempting unauthorized actions as the restricted user and confirming the system blocked the actions.

### A. Denial of Write Access (Least Privilege Proof)

The system returned an **ACCESS DENIED** error when the user attempted to upload a file to the data bucket. This proves the **Explicit DENY** in the Resource Policy successfully blocked the dangerous `s3:PutObject` action.

### B. Denial of Log Access (Separation of Duties Proof)

The system returned an **ACCESS DENIED** error when the user attempted to view the contents of the separate audit log bucket (`my-logs-ice-1`). This was achieved by an explicit **DENY policy** targeting the restricted user's identity on the log bucket.

-----

## 4\. 🚧 Troubleshooting and Key Learnings

### Challenge: Denial Override Failure

  * **Problem:** Initial policies failed to enforce security (user could still upload and see logs) because a global, unseen **"Allow"** permission was being inherited from the environment.
  * **Resolution:** This challenge was solved by implementing **Explicit DENY statements** in the **Resource Policy**. An explicit DENY statement on the resource has the highest authority in the AWS policy structure, forcing the architecture to respect the security rule over the inherited "Allow," a key learning in cloud policy evaluation.

### Challenge: Tool Limitation

  * **Problem:** Advanced debugging tools, such as the IAM Policy Simulator, were restricted in the training environment.
  * **Resolution:** Validation was achieved entirely through **direct live console testing** and layered **resource-based denial logic**, proving the ability to successfully secure a system even when relying on foundational tools.

-----

## 🔗 Code Artifacts (View Policies)

The specific policies used to enforce these rules can be found in the following files in this repository:

  * **User Policy:** `least-privilege-s3-readonly.json`
  * **Data Bucket Policy (Final Guardrail):** `final-secure-bucket-policy.json`
  * **Log Deny Policy (Fix):** `log-bucket-explicit-deny.json`
