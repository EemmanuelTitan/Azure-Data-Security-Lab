# Azure-Data-Security-Lab
Azure Data Security Lab Documentation

Overview
This document outlines the Azure data security configurations completed as part of a practical security exercise using an Azure for Students subscription.
Subscription: Azure for Students
Resource Group: My-Resource-Group
Region: South Africa North

Task 1 - Azure Key Vault Creation
Resource Name: kv-seclab-emmanuel
Pricing Tier: Standard
Location: South Africa North
Configuration:

Soft-delete: Enabled (90 day recovery window)
Purge protection: Disabled
Permission model: Azure RBAC
Role assigned: Key Vault Administrator

Risk Mitigated:
Without a Key Vault, secrets like passwords and API keys are often hardcoded in application code or stored in plain text config files. Azure Key Vault centralizes secrets management in a secure, audited location. This reduces the risk of credential exposure through source code leaks or unauthorized file access.

Task 2 - Storing a Secret
Secret Name: db-password
Value: Stored securely (not displayed)
Configuration:

Upload method: Manual
Expiry: None set (lab purposes)

Risk Mitigated:
Hardcoded database passwords in application code are one of the most common causes of data breaches. Storing the password in Key Vault means the application retrieves it dynamically at runtime — the password never appears in code, config files, or version control systems like GitHub.

Task 3 - Storing a Certificate
Certificate Name: lab-cert
Type: Self-signed
Subject: CN=Emmanuel
Validity: 12 months
Configuration:

Created and stored directly in Key Vault
Managed via Key Vault certificate lifecycle management

Risk Mitigated:
Certificates stored in files or on local machines can be stolen, lost, or forgotten and left to expire. Storing them in Key Vault ensures they are centrally managed, access-controlled, and auditable. This reduces the risk of certificate theft and service outages caused by unmanaged certificate expiry.

Task 4 - Azure Storage Account Creation
Resource Name: stseclabemmanuel02
Location: South Africa North
Performance: Standard
Redundancy: LRS (Locally Redundant Storage)
Account Kind: General Purpose v2
Risk Mitigated:
A storage account without security controls is a major data exposure risk. Using General Purpose v2 with security configurations applied ensures the account supports all modern security features including CMK encryption and HTTPS enforcement.

Task 5 - Disable Public Blob Access
Setting: Allow enabling anonymous access on individual containers → Disabled
Configuration:

Configured during storage account creation under the Security tab
Prevents any container from being set to anonymous/public access

Risk Mitigated:
By default, Azure storage containers can be configured to allow public anonymous access — meaning anyone on the internet with the URL can read your files without any authentication. Disabling this setting prevents accidental or intentional exposure of sensitive data to the public internet.

Task 6 - Enforce HTTPS Only
Setting: Require secure transfer for REST API operations → Enabled
Additional Setting: Require Encryption in Transit for SMB → Enabled
Configuration:

Configured during storage account creation under the Security tab
All HTTP requests are automatically rejected
Only HTTPS connections are accepted

Risk Mitigated:
HTTP connections transmit data in plain text, making them vulnerable to man-in-the-middle (MITM) attacks where an attacker intercepts and reads data in transit. Enforcing HTTPS ensures all data is encrypted during transmission using TLS, preventing eavesdropping and tampering.

Task 7 - Customer-Managed Key (CMK) Encryption
Key Vault: kv-seclab-emmanuel
Key Name: storage-cmk-key
Key Type: RSA 2048-bit
Identity Type: System-assigned managed identity
Scope: Blobs and files
Configuration:

RSA 2048-bit key generated in Key Vault
Storage account encryption type changed from Microsoft-managed keys (MMK) to Customer-managed keys (CMK)
Storage account granted access to Key Vault via system-assigned managed identity
Soft-delete and purge protection automatically enabled on Key Vault upon CMK activation

Risk Mitigated:
With Microsoft-managed keys, Microsoft controls the encryption keys for your data. If Microsoft's systems are compromised or you have compliance requirements, this is a risk. CMK gives you full control — if you revoke or delete the key, the data becomes inaccessible immediately. This satisfies regulatory requirements like GDPR, HIPAA and ISO 27001 that require customer control over encryption keys.

Task 8 - SAS Token with Limited Permissions
Token Type: Shared Access Signature (Account-level)
Service: File
Resource Types: Object only
Permissions: Read only
Protocol: HTTPS only
Start: 10 May 2026
Expiry: 11 May 2026 (24 hours)
Configuration:

Generated from Storage Account → Shared Access Signature
Strictly scoped to minimum required permissions
Short expiry window of 24 hours
HTTPS only enforced on the token
sv=2025-11-05&ss=f&srt=o&sp=r&se=2026-05-10T23:24:41Z&st=2026-05-10T15:09:41Z&spr=https&sig=uT33jWtUXyRNYNXH7sWjfIkC16bnhAdpTos2dUQP1I0%3D

Risk Mitigated:
Sharing full storage account access keys grants unlimited, permanent access to all data. A SAS token implements the principle of least privilege - the recipient can only perform the specific action they need (Read), on the specific resource type (Object), for a limited time (24 hours). Even if the token is stolen or leaked, the attacker can only read files and the token expires automatically within 24 hours rendering it useless.
