# Failures & Troubleshooting — Microsoft Entra ID Lab

## Issue 001 — Microsoft 365 Developer Program sandbox not available
**What happened:** Attempted to sign up for the free M365 Developer Program sandbox at developer.microsoft.com. Account was accepted but sandbox subscription was denied.
**Error:** "You don't currently qualify for a Microsoft 365 Developer Program sandbox subscription."
**Root cause:** Microsoft tightened eligibility in 2024-2025 — now requires active Visual Studio subscription or proven developer activity to qualify.
**Fix:** Used Microsoft Azure free account instead ($200 credits). Accessed Entra ID directly through portal.azure.com.
**Lesson:** Always have a backup path. Azure free account gives full Entra ID access without needing the M365 developer sandbox.

## Issue 002 — Entra admin center blocked personal Microsoft account
**What happened:** Tried accessing entra.microsoft.com directly with jovianney@ymail.com. Got error AADSTS50020 — account doesn't exist in the target tenant.
**Error:** "User account does not exist in tenant 'Microsoft Services' and cannot access the application."
**Root cause:** Personal Microsoft accounts (ymail, gmail, outlook.com) can't access the Entra admin center directly — it requires a work/school account or an account that IS the tenant admin.
**Fix:** Accessed Entra ID through Azure portal (portal.azure.com) which supports personal Microsoft accounts as tenant owners.
**Lesson:** Always access Entra ID through portal.azure.com when using a personal Microsoft account. The standalone entra.microsoft.com requires an organizational account.

## Issue 003 — Free tier Entra ID limited access
**What happened:** Some Entra ID sections returned 401/403 errors even as Global Administrator on the free tier.
**Error:** "You don't have access" on certain blades including Roles & admins in the standalone Entra admin center.
**Root cause:** Microsoft Entra ID Free tier restricts access to certain management features that require P1 or P2 licensing (Conditional Access, PIM, Identity Protection).
**Fix:** Used Azure portal instead of standalone Entra admin center — full user management available there on free tier.
**Lesson:** For full Entra ID lab work, use portal.azure.com → Microsoft Entra ID. The standalone admin center at entra.microsoft.com has more restrictions on free accounts.