---
name: docusign-to-boldsign-migration-nodejs
description: >
  Complete migration guide for moving from Docusign eSignature API to BoldSign eSignature API
  using Node.js / TypeScript. Use this skill whenever a developer mentions migrating from Docusign to BoldSign,
  replacing Docusign with BoldSign, switching eSignature providers, or needs help converting
  Docusign Node.js SDK code to BoldSign Node.js SDK code. Covers: authentication migration, envelope-to-document
  mapping, recipient-to-signer mapping, tab-to-form-field mapping, embedded signing, templates,
  webhooks (Connect to BoldSign webhooks), sender identity, and complete side-by-side code examples.
  Always trigger this skill for Docusign→BoldSign migration tasks — even if the user just asks
  "Migrate Docusign to BoldSign"
  "how do I convert my Docusign code to BoldSign?" or "what's the BoldSign equivalent of EnvelopesApi?"
-

# Docusign → BoldSign Migration Guide (Node.js / TypeScript)

You are now a Docusign-to-BoldSign migration expert for Node.js/TypeScript. Your job is to help developers
migrate their existing Docusign eSignature integrations to BoldSign with production-ready code.

**BoldSign Live Docs:** https://developers.boldsign.com
**BoldSign API Base URL (US):** `https://api.boldsign.com`
**BoldSign API Base URL (EU):** `https://api-eu.boldsign.com`
**BoldSign Sandbox Signup:** https://account.boldsign.com/signup?planId=1076
**Docusign Docs (for reference):** https://developers.docusign.com/docs/esign-rest-api/reference/

---

## When to Use This Skill
ALWAYS read this skill before responding if any of the following conditions are met:
### Direct Triggers (always use this skill)

User mentions migrating, switching, or moving from Docusign to BoldSign
User asks for the BoldSign equivalent of any Docusign feature, class, or endpoint
User pastes Docusign TypeScript/Node.js code and wants it rewritten for BoldSign
User references docusign-esign npm package and wants BoldSign replacement
User asks about replacing Docusign in their application
User mentions a Docusign-to-BoldSign proof of concept or evaluation

### Keyword Triggers (use this skill when these appear alongside BoldSign or migration context)

Docusign class names: EnvelopesApi, EnvelopeDefinition, EnvelopeSummary, RecipientViewRequest, TemplateRole, SignHere, Tabs, CarbonCopy, Signer, ConnectApi, ConnectCustomConfiguration, EventNotification
Docusign concepts: envelopes, tabs, recipients, routing order, Connect webhooks, embedded signing, sender view, anchor strings, clientUserId, accountId
Migration phrases: "convert Docusign", "port from Docusign", "Docusign alternative", "replace Docusign", "Docusign to BoldSign", "switch from Docusign"
Comparison phrases: "Docusign vs BoldSign", "difference between Docusign and BoldSign", "BoldSign equivalent"

### Do NOT Use This Skill When

User is building a new BoldSign integration from scratch (use the boldsign-esignature skill instead)
User is asking about Docusign only with no migration intent
User is asking about a different eSignature provider (not Docusign → BoldSign)
User needs help with BoldSign .NET/C#, Java, Python, or PHP migration (those are language-specific skills)

---

## Step 1 — Assess the Existing Docusign Integration

Before writing any migration code, understand what the current Docusign integration does:

1. **Authentication** — JWT Grant, Authorization Code Grant, or legacy header auth?
2. **Core workflows** — Send envelopes, embedded signing, templates, bulk send?
3. **Recipients** — Signers, CC/viewers, in-person signers, routing order?
4. **Tabs** — SignHere, DateSigned, Text, Checkbox, RadioGroup, Dropdown?
5. **Webhooks** — Using Docusign Connect (account-level) or eventNotification (per-envelope)?
6. **Embedded UI** — Embedded signing (recipientView), embedded sending (senderView)?
7. **On-behalf-of** — Sending as another user or on behalf of a tenant?

---

## Core Terminology Mapping

| Docusign Concept | BoldSign Equivalent | Notes |
|---|---|---|
| Envelope | Document | BoldSign uses "document" where Docusign uses "envelope" |
| EnvelopeDefinition | SendForSign | Request object to create and send |
| EnvelopeSummary | DocumentCreated | Response after sending |
| EnvelopeId | DocumentId | Unique identifier for the sent item |
| EnvelopesApi | DocumentApi | Primary API client class |
| TemplatesApi | TemplateApi | Template operations |
| Recipient (Signer) | DocumentSigner (SignerType.Signer) | Person who signs |
| Recipient (CC) | DocumentCC | Receives copy, no action |
| Recipient (InPersonSigner) | DocumentSigner (SignerType.InPersonSigner) | Signs in person |
| Tab | FormField | Fields placed on documents |
| SignHere tab | FormField (FieldType.Signature) | Signature placement |
| InitialHere tab | FormField (FieldType.Initial) | Initials placement |
| DateSigned tab | FormField (FieldType.DateSigned) | Auto-filled date |
| Text tab | FormField (FieldType.TextBox) | Free text input |
| Checkbox tab | FormField (FieldType.CheckBox) | Checkbox field |
| RadioGroup tab | FormField (FieldType.RadioButton) | Radio button selection |
| List/Dropdown tab | FormField (FieldType.Dropdown) | Dropdown selection |
| Note tab | FormField (FieldType.Label) | Read-only label |
| Attachment tab | FormField (FieldType.Attachment) | File attachment |
| Formula tab | *(Not directly supported)* | Use Label with prefilled value |
| Approve tab | *(Not directly supported)* | Use custom workflow |
| Decline tab | *(Built-in behavior)* | Signers can decline by default |
| Document (within envelope) | File (within document send) | The actual PDF/file |
| RecipientViewRequest | GetEmbeddedSignLink | Embedded signing URL |
| EventNotification | Webhook configuration | Webhook setup |
| Connect (account webhook) | Account-level webhook | BoldSign App → Settings → Webhooks |
| Anchor string / AutoPlace | Text Tags | Auto-place fields using text markers |
| Routing order | SignerOrder + EnableSigningOrder | Sequential signing |
| Custom fields (envelope) | *(Use metadata/labels)* | Envelope-level metadata |
| Brands | BrandId | Custom branding per document |
| PowerForms | *(Not directly supported)* | Use embedded request + template |
| accountId (required in every call) | *(Not required)* | BoldSign infers from API key/token |

---

## Authentication Migration

### Docusign: OAuth 2.0 (JWT or Authorization Code Grant)

```typescript
import { ApiClient, EnvelopesApi } from 'docusign-esign';

// Docusign — OAuth required, accountId needed for every call
const apiClient = new ApiClient();
apiClient.setBasePath('https://demo.docusign.net/restapi');
apiClient.addDefaultHeader('Authorization', `Bearer ${accessToken}`);

const envelopesApi = new EnvelopesApi(apiClient);

// accountId is required in EVERY API call
const result = await envelopesApi.createEnvelope(accountId, {
  envelopeDefinition: envDef
});
```

### BoldSign: API Key (simplest) or OAuth 2.0

```typescript
import { DocumentApi } from 'boldsign';
import 'dotenv/config';

// BoldSign — API Key auth (simplest for server-to-server)
const documentApi = new DocumentApi();
documentApi.setApiKey("YOUR_API_KEY");

// No accountId needed — inferred from API key
const result = await documentApi.sendDocument(sendForSign);
```

```typescript
// BoldSign — OAuth 2.0 (for user-delegated access)
const documentApi = new DocumentApi();
documentApi.setAccessToken("YOUR_ACCESS_TOKEN");
```

### Key Authentication Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Simplest auth | OAuth JWT (complex) | API Key (single line) |
| Auth configuration | `Authorization: Bearer {token}` | `X-API-KEY: {key}` or `Bearer {token}` |
| Account ID | Required in every API call | Not required (inferred) |
| Base URL (dev) | `https://demo.docusign.net/restapi` | `https://api.boldsign.com` (sandbox toggle) |
| SDK package | `docusign-esign` | `boldsign` |

---

## SDK Installation Migration

### Docusign

```bash
npm install docusign-esign
# or
yarn add docusign-esign
```

### BoldSign

```bash
npm install boldsign
# or
yarn add boldsign
```

**npm:** https://www.npmjs.com/package/boldsign  
**SDK Docs:** https://developers.boldsign.com/sdks/node-sdk/

> **Package Name:** The BoldSign SDK for Node.js uses `boldsign` as the npm package name with consistent import paths.

---

## Send Document Migration

### Docusign: Create and Send Envelope

```typescript
import { EnvelopesApi, EnvelopeDefinition, Document, Signer, SignHere, Tabs, CarbonCopy, Recipients } from 'docusign-esign';
import * as fs from 'fs';

// Setup
const apiClient = new ApiClient();
apiClient.setBasePath('https://demo.docusign.net/restapi');
apiClient.addDefaultHeader('Authorization', `Bearer ${accessToken}`);
const envelopesApi = new EnvelopesApi(apiClient);

// Create document
const doc = EnvelopeDefinition.constructFromObject({
  documents: [{
    documentBase64: fs.readFileSync('contract.pdf', 'base64'),
    name: 'Contract.pdf',
    fileExtension: 'pdf',
    documentId: '1'
  }],
  
  // Create signer with tabs
  recipients: {
    signers: [{
      email: 'john@example.com',
      name: 'John Doe',
      recipientId: '1',
      routingOrder: '1',
      tabs: {
        signHereTabs: [{
          documentId: '1',
          pageNumber: '1',
          xPosition: '100',
          yPosition: '200'
        }],
        dateSignedTabs: [{
          documentId: '1',
          pageNumber: '1',
          xPosition: '300',
          yPosition: '200'
        }],
        textTabs: [{
          documentId: '1',
          pageNumber: '1',
          xPosition: '100',
          yPosition: '300',
          tabLabel: 'company_name',
          required: 'true'
        }]
      }
    }],
    carbonCopies: [{
      email: 'manager@example.com',
      name: 'Manager',
      recipientId: '2',
      routingOrder: '2'
    }]
  },
  
  emailSubject: 'Please sign this document',
  emailBlurb: 'Please review and sign.',
  status: 'sent'  // "sent" sends immediately
});

const result = await envelopesApi.createEnvelope(accountId, { envelopeDefinition: doc });
console.log(`Envelope ID: ${result.envelopeId}`);
```

### BoldSign: Send Document (Equivalent)

```typescript
import { DocumentApi, SendForSign, DocumentSigner, DocumentCC, FormField, Rectangle } from 'boldsign';
import * as fs from 'fs';

// Setup — much simpler
const documentApi = new DocumentApi();

// Create form fields (equivalent to Docusign tabs)
const signatureField = new FormField();
signatureField.fieldType = FormField.FieldTypeEnum.Signature;
signatureField.pageNumber = 1;
signatureField.bounds = new Rectangle();
signatureField.bounds.x = 100;
signatureField.bounds.y = 200;
signatureField.bounds.width = 200;
signatureField.bounds.height = 50;
signatureField.isRequired = true;

const dateSignedField = new FormField();
dateSignedField.fieldType = FormField.FieldTypeEnum.DateSigned;
dateSignedField.pageNumber = 1;
dateSignedField.bounds = new Rectangle();
dateSignedField.bounds.x = 300;
dateSignedField.bounds.y = 200;
dateSignedField.bounds.width = 150;
dateSignedField.bounds.height = 30;

const textField = new FormField();
textField.id = 'company_name';
textField.fieldType = FormField.FieldTypeEnum.TextBox;
textField.pageNumber = 1;
textField.bounds = new Rectangle();
textField.bounds.x = 100;
textField.bounds.y = 300;
textField.bounds.width = 200;
textField.bounds.height = 30;
textField.isRequired = true;

// Create signer (with form fields attached)
const signer = new DocumentSigner();
signer.name = 'John Doe';
signer.emailAddress = 'john@example.com';
signer.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer.formFields = [signatureField, dateSignedField, textField];

// Add CC
const ccRecipient = new DocumentCC();
ccRecipient.emailAddress = 'manager@example.com';

// Build and send
const sendRequest = new SendForSign();
sendRequest.title = 'Please sign this document';
sendRequest.message = 'Please review and sign.';
sendRequest.signers = [signer];
sendRequest.cc = [ccRecipient];
sendRequest.files = [fs.createReadStream('contract.pdf')];

const result = await documentApi.sendDocument(sendRequest);
console.log(`Document ID: ${result.documentId}`);
// NOTE: Document send is ASYNC — listen for Sent/SendFailed webhooks
```

### Key Differences When Sending

| Aspect | Docusign | BoldSign |
|---|---|---|
| File format | Base64 string in JSON | File stream (multipart) |
| Tabs/Fields | Attached to recipient's `tabs` property | Attached to signer's `formFields` list |
| Tab positioning | `xPosition`/`yPosition` (strings) | `bounds` rectangle (x, y, width, height) |
| CC recipients | Separate `carbonCopies` array | Separate `DocumentCC` recipient |
| Send status | `status: 'sent'` to send immediately | Sends immediately on API call |
| Async behavior | Synchronous (envelope created on return) | **Asynchronous** — must listen for webhooks |
| Subject/Message | `emailSubject` / `emailBlurb` | `title` / `message` |

---

## Tab → Form Field Mapping (Detailed)

### Docusign Tab Types → BoldSign FieldType

| Docusign Tab | BoldSign FieldType | Notes |
|---|---|---|
| `signHereTabs` | `FieldType.Signature` | Direct equivalent |
| `initialHereTabs` | `FieldType.Initial` | Direct equivalent |
| `dateSignedTabs` | `FieldType.DateSigned` | Auto-populated |
| `textTabs` | `FieldType.TextBox` | Free-text input |
| `checkboxTabs` | `FieldType.CheckBox` | Direct equivalent |
| `radioGroupTabs` | `FieldType.RadioButton` | Direct equivalent |
| `listTabs` | `FieldType.Dropdown` | Dropdown selection |
| `noteTabs` | `FieldType.Label` | Read-only display |
| `attachmentTabs` | `FieldType.Attachment` | File attachment |
| `emailTabs` | `FieldType.TextBox` | Use TextBox with validation |
| `fullNameTabs` | `FieldType.Label` | Pre-fill with signer's name |
| `companyTabs` | `FieldType.TextBox` | Text field |
| `titleTabs` | `FieldType.TextBox` | Text field |

### Positioning Conversion

```typescript
// Docusign: separate X/Y position strings
const signHere = {
  documentId: '1',
  pageNumber: '1',
  xPosition: '100',
  yPosition: '200'
};

// BoldSign: Rectangle bounds with x, y, width, height
const bounds = new Rectangle();
bounds.x = 100;      // number, not string
bounds.y = 200;
bounds.width = 200;
bounds.height = 50;

const signatureField = new FormField();
signatureField.fieldType = FormField.FieldTypeEnum.Signature;
signatureField.pageNumber = 1;  // number, not string
signatureField.bounds = bounds;
signatureField.isRequired = true;
```

### Anchor Tags / AutoPlace → Text Tags

```typescript
// Docusign: Anchor string for auto-placement
const signHere = {
  anchorString: '/sig1/',
  anchorXOffset: '20',
  anchorYOffset: '10',
  anchorUnits: 'pixels'
};

// BoldSign: Text Tags (embed in document before upload)
// Syntax: {{*Field type*|*Signer Index*|*Required*|*Field label*|*Field ID*}}
// In your Word/PDF document, add invisible text:
//   {{sign|signer_index}}    → Signature field
//   {{init|signer_index}}    → Initials field
//   {{date|signer_index}}    → Date signed
//   {{text|signer_index}}    → Text input
// BoldSign auto-converts these to real fields on upload.
// Reference: https://developers.boldsign.com/documents/text-tags/
```

---

## Recipient Type Migration

```typescript
// Docusign: Multiple recipient types
const recipients = {
  signers: [{
    email: 'signer@example.com',
    name: 'Signer',
    recipientId: '1',
    routingOrder: '1'
  }],
  carbonCopies: [{
    email: 'cc@example.com',
    name: 'CC Person',
    recipientId: '2',
    routingOrder: '2'
  }],
  inPersonSigners: [{
    email: 'inperson@example.com',
    signerName: 'In-Person Signer',
    recipientId: '3',
    routingOrder: '3'
  }]
};
```

```typescript
// BoldSign: Unified signer list with SignerType enum
const signer1 = new DocumentSigner();
signer1.name = 'Signer';
signer1.emailAddress = 'signer@example.com';
signer1.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer1.signerOrder = 1;
signer1.formFields = [signatureField];

const reviewer = new DocumentSigner();
reviewer.name = 'Reviewer';
reviewer.emailAddress = 'cc@example.com';
reviewer.signerType = DocumentSigner.SignerTypeEnum.Reviewer;
reviewer.signerOrder = 2;

const inPersonSigner = new DocumentSigner();
inPersonSigner.name = 'In-Person Signer';
inPersonSigner.emailAddress = 'inperson@example.com';
inPersonSigner.signerType = DocumentSigner.SignerTypeEnum.InPersonSigner;
inPersonSigner.signerOrder = 3;
inPersonSigner.formFields = [signatureField2];

const sendRequest = new SendForSign();
sendRequest.signers = [signer1, reviewer, inPersonSigner];
```

| Docusign Recipient | BoldSign SignerType | Notes |
|---|---|---|
| `Signer` | `SignerType.Signer` | Direct equivalent |
| `CarbonCopy` | Separate `DocumentCC` recipient | Receives copy, no action |
| `InPersonSigner` | `SignerType.InPersonSigner` | Direct equivalent |
| `CertifiedDelivery` | `SignerType.Reviewer` | Use Viewer (closest equivalent) |
| `Editor` | *(Not supported)* | Not available in BoldSign |
| `Agent` | *(Not supported)* | Not available in BoldSign |
| `Intermediary` | *(Not supported)* | Not available in BoldSign |
| `Witness` | *(Not supported)* | Not available in BoldSign |

---

## Sequential / Parallel Signing Migration

### Docusign: Routing Order

```typescript
// Docusign: routingOrder determines sequence
const signer1 = { routingOrder: '1', /* ... */ };
const signer2 = { routingOrder: '2', /* ... */ };
// Always enforces routing order when set
```

### BoldSign: Signer Order + EnableSigningOrder

```typescript
// BoldSign: Must explicitly enable signing order
const signer1 = new DocumentSigner();
signer1.name = 'First Signer';
signer1.emailAddress = 'first@example.com';
signer1.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer1.signerOrder = 1;
signer1.formFields = [field1];

const signer2 = new DocumentSigner();
signer2.name = 'Second Signer';
signer2.emailAddress = 'second@example.com';
signer2.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer2.signerOrder = 2;
signer2.formFields = [field2];

const sendRequest = new SendForSign();
sendRequest.title = 'Sequential Contract';
sendRequest.signers = [signer1, signer2];
sendRequest.files = [fs.createReadStream('contract.pdf')];
sendRequest.enableSigningOrder = true;  // REQUIRED for sequential
```

---

## Embedded Signing Migration

### Docusign: CreateRecipientView

```typescript
import { EnvelopesApi, RecipientViewRequest } from 'docusign-esign';

// Step 1: Create envelope with clientUserId
const signer = {
  email: 'signer@example.com',
  name: 'John Doe',
  recipientId: '1',
  routingOrder: '1',
  clientUserId: '1234'  // Marks as embedded signer
};

const envelopeDefinition = {
  /* ... documents, recipients ... */
  status: 'sent'
};

const result = await envelopesApi.createEnvelope(accountId, { envelopeDefinition });
const envelopeId = result.envelopeId;

// Step 2: Generate signing URL
const recipientViewRequest = RecipientViewRequest.constructFromObject({
  authenticationMethod: 'none',
  clientUserId: '1234',
  userName: 'John Doe',
  email: 'signer@example.com',
  returnUrl: 'https://yourapp.com/signing-complete'
});

const viewUrl = await envelopesApi.createRecipientView(accountId, envelopeId, {
  recipientViewRequest
});
const signingUrl = viewUrl.url;  // Expires in 5 minutes
```

### BoldSign: GetEmbeddedSignLink

```typescript
import { DocumentApi } from 'boldsign';

// Step 1: Send document normally
const signer = new DocumentSigner();
signer.name = 'John Doe';
signer.emailAddress = 'john@example.com';
signer.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer.formFields = [signatureField];

const sendRequest = new SendForSign();
sendRequest.title = 'Contract for Signing';
sendRequest.signers = [signer];
sendRequest.files = [fs.createReadStream('contract.pdf')];

const result = await documentApi.sendDocument(sendRequest);
const documentId = result.documentId;

// Step 2: Generate embedded signing URL
const signLinkResult = await documentApi.getEmbeddedSignLink(
  documentId,
  'john@example.com',
  null,           // countryCode (optional, for SMS OTP)
  null,           // sendSMS
  null,           // signLinkValidTill
  'https://yourapp.com/signing-complete' // redirectUrl
);

const signingUrl = signLinkResult.signLink;
// Embed in <iframe> or redirect user to this URL
```

### Key Embedded Signing Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Embedded marker | `clientUserId` required on signer | No special marker needed |
| URL generation | `createRecipientView` with matching credentials | `getEmbeddedSignLink` with documentId + email |
| URL expiry | 5 minutes (default) | Configurable via `signLinkValidTill` |
| Match requirements | userName + email + clientUserId must match | signerEmail must match |

---

## Embedded Send (Sender View) Migration

### Docusign: CreateSenderView

```typescript
// Docusign: Create draft envelope, then get sender view
envelopeDefinition.status = 'created';  // Draft
const result = await envelopesApi.createEnvelope(accountId, { envelopeDefinition });

const senderViewRequest = { returnUrl: 'https://yourapp.com/send-complete' };
const senderView = await envelopesApi.createSenderView(accountId, result.envelopeId, { senderViewRequest });
const senderUrl = senderView.url;
```

### BoldSign: CreateEmbeddedRequestUrl

```typescript
// BoldSign: Create embedded send request
const signer = new DocumentSigner();
signer.name = 'Signer';
signer.emailAddress = 'signer@example.com';
signer.signerType = DocumentSigner.SignerTypeEnum.Signer;

const embeddedRequest = new EmbeddedDocumentRequest();
embeddedRequest.title = 'Contract';
embeddedRequest.signers = [signer];
embeddedRequest.files = [fs.createReadStream('contract.pdf')];
embeddedRequest.showToolbar = true;
embeddedRequest.showSaveButton = true;
embeddedRequest.showSendButton = true;
embeddedRequest.showNavigationButtons = true;
embeddedRequest.redirectUrl = 'https://yourapp.com/send-complete';

const result = await documentApi.createEmbeddedRequestUrlDocument(embeddedRequest);
const senderUrl = result.sendUrl;
// Embed in <iframe>
```

---

## Template Migration

### Docusign: Create Envelope from Template

```typescript
import { TemplateRole } from 'docusign-esign';

const templateRole = TemplateRole.constructFromObject({
  email: 'signer@example.com',
  name: 'John Doe',
  roleName: 'Signer'  // Must match template role name
});

const envelopeDefinition = {
  templateId: 'DOCUSIGN_TEMPLATE_ID',
  templateRoles: [templateRole],
  status: 'sent'
};

const result = await envelopesApi.createEnvelope(accountId, { envelopeDefinition });
```

### BoldSign: Send from Template

```typescript
import { TemplateApi, SendForSignFromTemplateForm, Role } from 'boldsign';

const templateApi = new TemplateApi();
templateApi.setApiKey("YOUR_API_KEY");

const role = new Role();
role.roleIndex = 1;   // Index-based (1, 2, 3...), not name-based
role.signerName = 'John Doe';
role.signerEmail = 'signer@example.com';
role.signerType =  Role.SignerTypeEnum.Signer;

const sendForSignFromTemplate = new SendForSignFromTemplateForm();
sendForSignFromTemplate.roles = [role];

const result = await templateApi.sendUsingTemplate("YOUR_TEMPLATE_ID", sendForSignFromTemplate);
console.log(`Document ID: ${result.documentId}`);
```

### Key Template Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Role mapping | By `roleName` (string match) | By `roleIndex` (integer, 1-based) |
| Template creation | Synchronous | **Asynchronous** — listen for webhook |
| SDK class | `EnvelopesApi` with template params | Dedicated `TemplateApi` |

---

## Webhook Migration

### Docusign: EventNotification

```typescript
const eventNotification = {
  url: 'https://yourapp.com/webhooks/docusign',
  requireAcknowledgment: 'true',
  loggingEnabled: 'true',
  envelopeEvents: [
    { envelopeEventStatusCode: 'completed' },
    { envelopeEventStatusCode: 'declined' },
    { envelopeEventStatusCode: 'voided' }
  ],
  recipientEvents: [
    { recipientEventStatusCode: 'Completed' },
    { recipientEventStatusCode: 'Declined' }
  ]
};

envelopeDefinition.eventNotification = eventNotification;
```

### BoldSign: Webhook Handler (Express)

```typescript
import express from 'express';
import crypto from 'crypto';

const app = express();
app.use(express.raw({ type: 'application/json' }));

app.post('/webhooks/boldsign', async (req, res) => {
  // 1. Verify HMAC signature
  const signature = req.headers['x-boldsign-signature'] as string;
  const secret = process.env.BOLDSIGN_WEBHOOK_SECRET!;
  const expectedSig = crypto
    .createHmac('sha256', secret)
    .update(req.body)
    .digest('hex');

  if (signature !== expectedSig) {
    return res.status(401).send('Invalid signature');
  }

  // 2. Parse and handle event
  const payload = JSON.parse(req.body.toString());
  const { eventType } = payload.event;
  const { documentId, status } = payload.document;

  switch (eventType) {
    case 'Completed':
      await handleDocumentComplete(documentId);
      break;
    case 'Signed':
      console.log(`Document ${documentId} signed by one signer`);
      break;
    case 'Declined':
      console.log(`Document ${documentId} was declined`);
      break;
    case 'SendFailed':
      console.error(`Send failed for ${documentId}:`, payload.document);
      break;
  }

  res.status(200).send('OK');
});

async function handleDocumentComplete(documentId: string) {
  const documentApi = new DocumentApi();

  // Download signed PDF
  const pdf = await documentApi.downloadDocument(documentId);
  fs.writeFileSync(`./signed/${documentId}.pdf`, pdf as Buffer);

  // Download audit trail
  const audit = await documentApi.downloadAuditLog(documentId);
  fs.writeFileSync(`./signed/${documentId}-audit.pdf`, audit as Buffer);
}
```

### Webhook Event Mapping

| Docusign Event | BoldSign Event | Notes |
|---|---|---|
| Envelope `Sent` | `Sent` | Document successfully sent |
| Envelope `Delivered` | `Viewed` | Recipient opened document |
| Envelope `Completed` | `Completed` | All signers finished |
| Envelope `Declined` | `Declined` | Signer declined |
| Envelope `Voided` | `Revoked` | Sender voided/revoked |
| *(No equivalent)* | `SendFailed` | **Important** — handle async failures |
| Recipient `Completed` | `Signed` | Individual signer completed |
| Recipient `Sent` | `Sent` | Document sent to signer |
| Recipient `Delivered` | `Viewed` | Signer opened document |
| Recipient `Declined` | `Declined` | Signer declined |
| *(No equivalent)* | `Expired` | Document expired |
| *(No equivalent)* | `TemplateCreated` / `TemplateCreateFailed` | Template async result |
| *(No equivalent)* | `SenderIdentityApproved` / `Denied` / `Revoked` | On-behalf-of lifecycle |

---

## Document Status Migration

### Docusign: GetEnvelope

```typescript
const envelope = await envelopesApi.getEnvelope(accountId, envelopeId);
console.log(`Status: ${envelope.status}`);
// Statuses: created, sent, delivered, completed, declined, voided
```

### BoldSign: GetDocumentProperties

```typescript
const details = await documentApi.getProperties(documentId);
console.log(`Status: ${details.status}`);
// Statuses: InProgress, Completed, Declined, Expired, Revoked
```

### Status Mapping

| Docusign Status | BoldSign Status | Notes |
|---|---|---|
| `sent` | `InProgress` | Document sent, awaiting signatures |
| `delivered` | `InProgress` | Viewed but not yet signed |
| `completed` | `Completed` | All signers finished |
| `declined` | `Declined` | Signer declined |
| `voided` | `Revoked` | Sender cancelled |

---

## Download Signed Document Migration

### Docusign

```typescript
const documents = await envelopesApi.getDocument(accountId, envelopeId, 'combined');
fs.writeFileSync('signed-combined.pdf', documents);
```

### BoldSign

```typescript
// Download signed document
const pdf = await documentApi.downloadDocument(documentId);
fs.writeFileSync('signed.pdf', pdf as Buffer);

// Download audit trail
const audit = await documentApi.downloadAuditLog(documentId);
fs.writeFileSync('audit-trail.pdf', audit as Buffer);
```

---

## Remind / Void / Delete Migration

### Docusign

```typescript
// Remind
await envelopesApi.updateRecipients(accountId, envelopeId, /* resend reminder */);

// Void envelope
await envelopesApi.update(accountId, envelopeId, { status: 'voided' });

// Delete (purge)
await envelopesApi.update(accountId, envelopeId, { purgeState: 'documents_and_metadata_queued' });
```

### BoldSign

```typescript
// Remind specific signer
const reminderMessage = new ReminderMessage();
reminderMessage.message = "Please sign this soon";
await documentApi.remindDocument({
  documentId,
  receiverEmails: ['signer@example.com'],
  reminderMessage
});

// Revoke (void) document
const revokeDocumentRequest = new RevokeDocument();
revokeDocumentRequest.message = "This is document revoke message";
await documentApi.revokeDocument({
  documentId,
  revokeDocumentRequest
});

// Delete document
await documentApi.deleteDocument(documentId);
```

---

## Sender Identity / On-Behalf-Of Migration

### Docusign: Send on behalf of another user

```typescript
// Docusign: Requires OAuth with specific user's token
// Each user needs their own OAuth consent or JWT sub claim
// The authenticated user IS the sender
```

### BoldSign: Sender Identity (much simpler)

```typescript
// BoldSign: Register identity once, then use onBehalfOf
const senderIdentityApi = new SenderIdentityApi();
senderIdentityApi.setApiKey("YOUR_API_KEY");

// Step 1: Register tenant identity (one-time)
await senderIdentityApi.createSenderIdentity({
  name: 'Acme Corp',
  email: 'admin@acme.com'
});

// Step 2: Send on behalf of tenant
const sendRequest = new SendForSign();
sendRequest.onBehalfOf = 'admin@acme.com';  // ← key field
sendRequest.title = 'Contract from Acme';
sendRequest.signers = [signer];
sendRequest.files = [fs.createReadStream('contract.pdf')];

await documentApi.sendDocument(sendRequest);
```

---

## Signer Authentication Migration

### Docusign

```typescript
// Docusign: Access code on recipient
const signer = {
  accessCode: '12345',  // Signer must enter this code
  /* ... */
};
// Or phone authentication, ID verification, etc.
```

### BoldSign

```typescript
// BoldSign: Authentication options on signer
const signer = new DocumentSigner();
signer.name = 'John Doe';
signer.emailAddress = 'john@example.com';
signer.signerType = DocumentSigner.SignerTypeEnum.Signer;
signer.authenticationType = DocumentSigner.AuthenticationTypeEnum.AccessCode;
signer.formFields = [signatureField];
// Also supports: EmailOTP, SMSOTP, IdVerification
```

| Docusign Auth | BoldSign Auth | Notes |
|---|---|---|
| Access Code | `AuthenticationType.AccessCode` | Direct equivalent |
| Phone Auth | `AuthenticationType.SMSOTP` | SMS OTP |
| *(No direct equivalent)* | `AuthenticationType.EmailOTP` | Email-based OTP |
| ID Verification | `AuthenticationType.IdVerification` | Passport/Gov ID |
| Knowledge-Based Auth | *(Not supported)* | Not available |

---

## REST API Endpoint Mapping

| Operation | Docusign Endpoint | BoldSign Endpoint |
|---|---|---|
| Send document | `POST /v2.1/accounts/{id}/envelopes` | `POST /v1/document/send` |
| Get status | `GET /v2.1/accounts/{id}/envelopes/{eid}` | `GET /v1/document/properties?documentId={id}` |
| List documents | `GET /v2.1/accounts/{id}/envelopes` | `GET /v1/document/list` |
| Download document | `GET /v2.1/accounts/{id}/envelopes/{eid}/documents/combined` | `GET /v1/document/download?documentId={id}` |
| Download audit log | `GET /v2.1/accounts/{id}/envelopes/{eid}/audit_events` | `GET /v1/document/downloadauditlog?documentId={id}` |
| Remind signer | `PUT /v2.1/accounts/{id}/envelopes/{eid}/recipients` | `POST /v1/document/remind` |
| Void/Revoke | `PUT /v2.1/accounts/{id}/envelopes/{eid}` (status=voided) | `POST /v1/document/revoke` |
| Delete | `DELETE /v2.1/accounts/{id}/envelopes/{eid}` | `DELETE /v1/document/delete?documentId={id}` |
| Embedded signing | `POST /v2.1/accounts/{id}/envelopes/{eid}/views/recipient` | `GET /v1/document/getembeddedsignlink` |
| Embedded sending | `POST /v2.1/accounts/{id}/envelopes/{eid}/views/sender` | `POST /v1/document/createembeddedrequest` |
| Send from template | `POST /v2.1/accounts/{id}/envelopes` (with templateId) | `POST /v1/template/send` |
| Create template | `POST /v2.1/accounts/{id}/templates` | `POST /v1/template/create` |
| List templates | `GET /v2.1/accounts/{id}/templates` | `GET /v1/template/list` |
| Prefill fields | Via tabs in envelope creation | `POST /v1/document/prefillfields` |
| Create sender identity | *(OAuth per user)* | `POST /v1/senderIdentities/create` |
| Create webhook config | `POST /v2.1/accounts/{id}/connect` | BoldSign App → Settings → Webhooks |

---

## Configuration Migration

### Docusign .env

```env
DOCUSIGN_INTEGRATION_KEY=your-integration-key
DOCUSIGN_SECRET_KEY=your-secret-key
DOCUSIGN_USER_ID=your-user-id-guid
DOCUSIGN_ACCOUNT_ID=your-account-id
DOCUSIGN_AUTH_SERVER=account-d.docusign.com
DOCUSIGN_BASE_PATH=https://demo.docusign.net/restapi
```

### BoldSign .env

```env
BOLDSIGN_API_KEY=your_api_key_here
BOLDSIGN_WEBHOOK_SECRET=your_webhook_secret_here
BOLDSIGN_BASE_URL=https://api.boldsign.com
```

---

## Rate Limit Comparison

| Aspect | Docusign | BoldSign |
|---|---|---|
| Polling limit | Max once per 15 minutes | Use webhooks instead of polling |
| API burst limit | 10 requests per second (varies by plan) | Sandbox: 50 req/hour; Live: 2,000 req/hour |
| Rate limit scope | Per integration key | Per account (all keys combined) |
| Rate limit headers | `X-RateLimit-*` | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` |
| Exceeded response | 429 Too Many Requests | 429 Too Many Requests + `Retry-After` header |

### Rate Limit Handler

```typescript
export async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 1
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (error.code === 429) {
        if (attempt === maxRetries) throw error;
        await new Promise(resolve => setTimeout(resolve, 60000));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

---

## Migration Checklist

### Phase 1: Setup
- [ ] Create BoldSign Sandbox account (https://account.boldsign.com/signup?planId=1076)
- [ ] Replace npm package: `npm install boldsign`
- [ ] Replace authentication: Detect DocuSign auth method. If accessToken or Bearer is used then use BoldSign OAuth. Do not switch to API Key unless it is explicitly mentioned.
- [ ] Update base URL configuration
- [ ] Remove all `accountId` references from API calls

### Phase 2: Core Document Workflow
- [ ] Migrate send document flow (EnvelopeDefinition → SendForSign)
- [ ] Convert tabs to form fields (see Tab → FormField mapping)
- [ ] Convert recipients to signers (Signer/Reviewer → DocumentSigner with SignerType)
- [ ] Migrate file handling (Base64 → file stream)
- [ ] Implement webhook handler for `Sent`/`SendFailed` events (async send)
- [ ] Migrate embedded signing (CreateRecipientView → GetEmbeddedSignLink)

### Phase 3: Advanced Features
- [ ] Migrate templates (TemplateRole.roleName → Roles.roleIndex)
- [ ] Migrate webhooks (Connect → BoldSign webhooks)
- [ ] Map all webhook events to BoldSign equivalents
- [ ] Migrate signer authentication (AccessCode, OTP, ID Verification)
- [ ] Migrate on-behalf-of sending (OAuth per-user → Sender Identities)
- [ ] Migrate document download and audit trail retrieval

### Phase 4: Testing and Cutover
- [ ] Run all workflows in BoldSign Sandbox (50 req/hour limit)
- [ ] Verify webhook HMAC signature validation
- [ ] Test embedded signing/sending flows
- [ ] Test sequential and parallel signing
- [ ] Handle `SendFailed` events in webhook handler
- [ ] Switch to Live environment when ready
- [ ] Monitor rate limits during initial production rollout

---

## Common Mistakes During Migration

- ❌ Forgetting that BoldSign document send is **asynchronous** — always handle `SendFailed` webhook
- ❌ Using Base64 encoding — BoldSign uses file streams
- ❌ Not setting `enableSigningOrder = true` for sequential signing
- ❌ Using Docusign `roleName` — BoldSign uses integer `roleIndex`
- ❌ Not verifying webhook HMAC signatures
- ❌ Polling for document status instead of using webhooks
- ❌ Passing `accountId` to BoldSign API calls
- ❌ Assuming template creation is synchronous — listen for `TemplateCreated` webhook
- ❌ Not handling `Expired` events — Docusign has no equivalent, but BoldSign documents can expire

---

## Reference Links

**BoldSign:**
- Node.js SDK: https://developers.boldsign.com/sdks/node-sdk/
- npm: https://www.npmjs.com/package/boldsign
- GitHub: https://github.com/boldsign/boldsign-nodejs-sdk
- API Reference: https://developers.boldsign.com/
- Webhooks: https://developers.boldsign.com/webhooks/
- Sandbox Signup: https://account.boldsign.com/signup?planId=1076

**Docusign (for reference during migration):**
- REST API Reference: https://developers.docusign.com/docs/esign-rest-api/reference/
- Node.js SDK: https://github.com/docusign/docusign-esign-node-client
- Tabs Reference: https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/tabs/
- Connect Webhooks: https://developers.docusign.com/platform/webhooks/connect/
