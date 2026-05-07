# Day 15 — Azure Key Vault & Security Center

> Phase 4 — Identity, Security + Monitoring

## Goal

Store secrets securely in Azure Key Vault and understand Azure's security posture tools.

## Key Topics

- Azure Key Vault: a centralized, secure store for three types of sensitive data
  - **Secrets**: API keys, passwords, database connection strings — anything you'd otherwise hardcode
  - **Keys**: cryptographic keys for encryption/decryption (RSA, EC) — Key Vault can perform crypto operations on your behalf without ever exposing the raw key
  - **Certificates**: SSL/TLS certificates — create, import, auto-renew, and deploy from one place
- Access control for Key Vault:
  - Access Policies model: grant a user or app permission to specific secret/key/certificate operations
  - RBAC model (recommended): use standard Azure role assignments (Key Vault Secrets User, Key Vault Administrator, etc.)
- Soft delete: deleted secrets are retained for 7–90 days and can be recovered — protects against accidental deletion
- Purge protection: prevents permanent deletion during the soft-delete retention period — required for compliance scenarios
- Integrating Key Vault with apps: reference a Key Vault secret in App Service or Azure Functions using a Key Vault reference — the app gets the value at runtime without the secret ever being in your code or config files
- Microsoft Defender for Cloud (formerly Security Center): the unified security management platform across all your Azure resources
  - **Secure Score**: a percentage score that measures how well your environment follows security best practices — each recommendation raises your score when implemented
  - Security recommendations: specific, actionable steps to improve posture (e.g., "Enable MFA for accounts with owner permissions")
  - Regulatory compliance dashboard: shows your alignment with standards like PCI-DSS, ISO 27001, CIS
  - Microsoft Defender plans: enhanced threat detection for specific services (VMs, SQL, Storage, Containers) — priced per resource
- Microsoft Sentinel: cloud-native SIEM (Security Information and Event Management) — collects logs from across your environment and uses AI to detect threats and automate responses

## Hands-On Demo

**Account Requirements:** Key Vault Standard tier charges per operation (~$0.03/10,000 operations — negligible for learning). Defender for Cloud free tier shows basic Secure Score. Advanced Defender plans are paid — instructor will demonstrate.

- ✅ Create a Key Vault (Standard tier)
- ✅ Store a secret (e.g., a database connection string)
- ✅ Retrieve the secret from an App Service using a Managed Identity and Key Vault reference (no hardcoded credentials in code)
- 💳 Review Microsoft Defender for Cloud enhanced security recommendations and compliance dashboard (Defender paid plans — instructor demo; students can view the free Secure Score)

## Summary

Never hardcode secrets. Key Vault + Managed Identity is the correct pattern for apps accessing sensitive configuration in Azure. Defender for Cloud is your always-on security advisor — even the free Secure Score gives you a clear prioritized list of what to fix first.
