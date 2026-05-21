---
name: Docusign to BoldSign API Migration
description: >
  Use this skill whenever the user wants to migrate an existing Docusign eSignature
  integration to BoldSign. This includes converting envelopes to documents, recipients
  to signers, tabs to form fields, Connect webhooks to BoldSign webhooks, embedded signing
  and sending flows, templates, sender identities, authentication models, and overall
  workflow parity. Also use this skill when comparing Docusign vs BoldSign features or
  generating migration-specific code for Node.js/TypeScript, Java, .NET/C#, Python, or PHP.
---

# Docusign → BoldSign Migration Skill

This skill is the **entry point** for all Docusign-to-BoldSign migration scenarios.
It provides conceptual mapping, migration rules, pitfalls, and directs Agent
to the correct **stack-specific reference files** before generating any code.

**BoldSign Developer Docs:** https://developers.boldsign.com
**API Base URL (US):** https://api.boldsign.com
**API Base URL (EU):** https://api-eu.boldsign.com
**API Base URL (CA):** https://api-ca.boldsign.com
**API Base URL (AU):** https://api-au.boldsign.com

---

## When to Use This Skill

Always use this skill if the user:

- Mentions **Docusign → BoldSign migration**
- Wants to **replace Docusign with BoldSign**
- Asks for **BoldSign equivalents** of Docusign APIs or concepts
- Pastes Docusign code and asks to **convert or rewrite**
- Asks about **Docusign vs BoldSign**
- Mentions envelopes, recipients, tabs, Connect, embedded signing, sender view
- Is evaluating BoldSign as a **Docusign alternative**

### Do NOT use this skill when
- The user is building a **new BoldSign integration from scratch**
- The user asks only about Docusign (no migration intent)
- The user is migrating from any provider other than Docusign

---

## Required Clarifications (Before Coding)

Before generating code, always identify:

1. **Target stack**
   - Node.js / TypeScript
   - Java
   - .NET / C#
   - Python
   - PHP

2. **Current Docusign usage**
   - Envelopes or Templates
   - Embedded signing or email-based signing
   - Connect webhooks
   - OAuth (JWT / Auth Code)
   - Single-tenant or multi-tenant SaaS

3. **Target BoldSign auth model**
   - API Key (backend automation)
   - OAuth 2.0 (user-delegated)
   - Sender Identities (`onBehalfOf`) for SaaS

---

## Stack Reference Files (Read Before Writing Code)

| Stack | Reference File |
|---|---|
| Node.js / TypeScript | `references/stacks/nodejs.md` |
| Java | `references/stacks/java.md` |
| .NET / C# | `references/stacks/dotnet.md` |
| Python | `references/stacks/python.md` |
| PHP | `references/stacks/php.md` |

⚠️ These files contain **complete side-by-side migration code** and must be followed exactly.

---

## Core Concept Mapping

| Docusign | BoldSign |
|---|---|
| Envelope | Document |
| Recipient | Signer |
| Tabs | Form Fields |
| Routing Order | Signer Order + EnableSigningOrder |
| Connect | Webhooks |
| Embedded Signing (Recipient View) | Embedded Sign Link |
| Embedded Sending (Sender View) | Embedded Send Request |
| Template Roles (name-based) | Template Roles (index-based) |
| Integration Key / JWT | API Key or OAuth |
| User-based sending | Sender Identity |
| Anchor Strings | Text Tags |

---

## Critical Migration Rules ⚠️

- **BoldSign document sending is asynchronous**
  - Always rely on webhooks (`Sent`, `SendFailed`)
- **Never poll document status**
  - Replace polling with webhooks
- **Verify webhook HMAC (`X-BoldSign-Signature`)**
  - Required for production security
- **Do not pass `accountId`**
  - BoldSign infers account from API key/token
- **Use multipart file upload**
  - Base64 JSON uploads (Docusign style) are invalid
- **Templates and documents create async**
  - Listen for webhook confirmation

---

## Authentication Migration

| Docusign | BoldSign |
|---|---|
| OAuth JWT (server automation) | API Key |
| OAuth Auth Code (user apps) | OAuth 2.0 |
| Send as another user | Sender Identity (`onBehalfOf`) |

For SaaS migrations, **Sender Identities** are strongly recommended.

---

## Webhook Migration (Connect → BoldSign)

| Docusign Event | BoldSign Event |
|---|---|
| Envelope Sent | Sent |
| Envelope Completed | Completed |
| Recipient Completed | Signed |
| Envelope Declined | Declined |
| Envelope Voided | Revoked |
| Envelope Expired | Expired |

---

## Rate Limit Awareness

| Environment | Limit |
|---|---|
| Sandbox | 50 requests/hour |
| Live | 2,000 requests/hour |

Docusign-style status polling **will hit limits quickly**  
Always migrate to **event-driven webhooks**

---

## Recommended Migration Order

1. Analyze existing Docusign workflows
2. Replace envelopes → documents
3. Convert tabs → form fields or text tags
4. Replace polling → webhooks
5. Choose BoldSign auth model
6. Migrate embedded signing/sending
7. Recreate templates
8. Implement sandbox testing
9. Switch to live environment

---

## Common Migration Mistakes ❌

- Assuming document send is synchronous
- Using Base64 uploads
- Polling instead of webhooks
- Skipping HMAC verification
- Using template role names instead of role indexes
- Passing Docusign `accountId`
- Ignoring `SendFailed` events
- Testing aggressively in Sandbox

---

## How Agent Should Respond

- Use **this SKILL.md first**
- Then load the **correct stack reference**
- Generate **migration-focused answers only**
- Always explain **Docusign → BoldSign equivalence**
- Default to **best-practice migration patterns**