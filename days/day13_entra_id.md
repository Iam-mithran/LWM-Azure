# Day 13 — Microsoft Entra ID — Identity Foundation

> Phase 4 — Identity, Security + Monitoring

## Goal

Understand Azure's identity system — how users, groups, and services prove who they are in Azure.

## Key Topics

- What is Microsoft Entra ID (formerly Azure Active Directory)? Microsoft's cloud-based identity and access management service — it controls who can sign in and what they can access
- Key concepts:
  - Tenant: your organization's dedicated instance of Entra ID — created automatically when you sign up for Azure
  - Directory: the store of all users, groups, and app registrations in your tenant
  - Subscription: linked to a tenant — a subscription trusts one Entra ID tenant for authentication
- Users: identities that can sign in
  - Cloud-only users: created directly in Entra ID
  - Synced users: sourced from on-premises Active Directory via Azure AD Connect
- Groups: collections of users (and other groups) used to assign permissions in bulk
  - Security groups: used for RBAC and application access
  - Microsoft 365 groups: used for collaboration (Teams, SharePoint)
- Entra ID tiers: Free, Premium P1, Premium P2 (advanced features like Conditional Access and Identity Protection require Premium)
- Service Principals & App Registrations: how applications authenticate to Azure — register your app, get a client ID and secret, and your app can call Azure APIs
- Managed Identities: give an Azure resource (e.g., a VM or Function App) its own identity — no credentials to manage, no secrets to rotate
  - System-assigned: tied to the lifecycle of one resource
  - User-assigned: standalone identity that can be attached to multiple resources
- Multi-Factor Authentication (MFA): require a second proof of identity (app code, SMS, hardware key) at sign-in
- Self-Service Password Reset (SSPR): let users reset their own passwords without calling IT
- Conditional Access: policies that enforce requirements (MFA, compliant device, specific location) before granting access — requires Entra ID Premium P1

## Hands-On Demo

**Account Requirements:** Creating users, groups, MFA, and Managed Identities are all free tier. Conditional Access requires Entra ID Premium P1 (paid) — instructor will demonstrate.

- ✅ Create a user in Entra ID
- ✅ Create a security group and add the user to it
- ✅ Enable MFA for the user
- ✅ Create a system-assigned Managed Identity on a VM
- 💳 Configure a Conditional Access policy (requires Entra ID Premium P1 — instructor demo; students can view the blade and understand the settings)

## Summary

Entra ID is the identity backbone of Azure. Every sign-in, permission grant, and service-to-service call goes through it. Managed Identities are the right way to let Azure services authenticate to each other — no secrets, no rotation headaches.
