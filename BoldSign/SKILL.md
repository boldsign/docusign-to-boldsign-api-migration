---
name: docusign-to-boldsign-migration-dotnet
description: >
  Complete migration guide for moving from DocuSign eSignature API to BoldSign eSignature API
  using .NET / C#. Use this skill whenever a developer mentions migrating from DocuSign to BoldSign,
  replacing DocuSign with BoldSign, switching eSignature providers, or needs help converting
  DocuSign C# SDK code to BoldSign C# SDK code. Covers: authentication migration, envelope-to-document
  mapping, recipient-to-signer mapping, tab-to-form-field mapping, embedded signing, templates,
  webhooks (Connect to BoldSign webhooks), sender identity, and complete side-by-side code examples.
  Always trigger this skill for DocuSign→BoldSign migration tasks — even if the user just asks
  "Migrate Docusign to BoldSign"
  "how do I convert my DocuSign code to BoldSign?" or "what's the BoldSign equivalent of EnvelopesApi?"
---

# DocuSign → BoldSign Migration Guide (.NET / C#)

You are now a DocuSign-to-BoldSign migration expert for .NET/C#. Your job is to help developers
migrate their existing DocuSign eSignature integrations to BoldSign with production-ready code.

**BoldSign Live Docs:** https://developers.boldsign.com
**BoldSign API Base URL (US):** `https://api.boldsign.com`
**BoldSign API Base URL (EU):** `https://eu-api.boldsign.com`
**BoldSign Sandbox Signup:** https://account.boldsign.com/signup?planId=1076
**DocuSign Docs (for reference):** https://developers.docusign.com/docs/esign-rest-api/reference/

---

## When to Use This Skill
ALWAYS read this skill before responding if any of the following conditions are met:
### Direct Triggers (always use this skill)

User mentions migrating, switching, or moving from DocuSign to BoldSign
User asks for the BoldSign equivalent of any DocuSign feature, class, or endpoint
User pastes DocuSign C# code and wants it rewritten for BoldSign
User references DocuSign.eSign NuGet package and wants BoldSign.Api replacement
User asks about replacing DocuSign in their application
User mentions a DocuSign-to-BoldSign proof of concept or evaluation

### Keyword Triggers (use this skill when these appear alongside BoldSign or migration context)

DocuSign class names: EnvelopesApi, EnvelopeDefinition, EnvelopeSummary, RecipientViewRequest, TemplateRole, SignHere, Tabs, CarbonCopy, Signer, ConnectApi, ConnectCustomConfiguration, EventNotification
DocuSign concepts: envelopes, tabs, recipients, routing order, Connect webhooks, embedded signing, sender view, anchor strings, clientUserId, accountId
Migration phrases: "convert DocuSign", "port from DocuSign", "DocuSign alternative", "replace DocuSign", "DocuSign to BoldSign", "switch from DocuSign"
Comparison phrases: "DocuSign vs BoldSign", "difference between DocuSign and BoldSign", "BoldSign equivalent"

### Do NOT Use This Skill When

User is building a new BoldSign integration from scratch (use the boldsign-esignature skill instead)
User is asking about DocuSign only with no migration intent
User is asking about a different eSignature provider (not DocuSign → BoldSign)
User needs help with BoldSign Node.js, Python, or PHP migration (this skill is .NET/C# only)

---

## Step 1 — Assess the Existing DocuSign Integration

Before writing any migration code, understand what the current DocuSign integration does:

1. **Authentication** — JWT Grant, Authorization Code Grant, or legacy header auth?
2. **Core workflows** — Send envelopes, embedded signing, templates, bulk send?
3. **Recipients** — Signers, CC/viewers, in-person signers, routing order?
4. **Tabs** — SignHere, DateSigned, Text, Checkbox, RadioGroup, Dropdown?
5. **Webhooks** — Using DocuSign Connect (account-level) or eventNotification (per-envelope)?
6. **Embedded UI** — Embedded signing (recipientView), embedded sending (senderView)?
7. **On-behalf-of** — Sending as another user or on behalf of a tenant?

---

## Core Terminology Mapping

| DocuSign Concept | BoldSign Equivalent | Notes |
|---|---|---|
| Envelope | Document | BoldSign uses "document" where DocuSign uses "envelope" |
| EnvelopeDefinition | SendForSign | Request object to create and send |
| EnvelopeSummary | DocumentCreated | Response after sending |
| EnvelopeId | DocumentId | Unique identifier for the sent item |
| EnvelopesApi | DocumentClient | Primary API client class |
| TemplatesApi | TemplateClient | Template operations |
| Recipient (Signer) | DocumentSigner (SignerType.Signer) | Person who signs |
| Recipient (CC) | DocumentSigner (SignerType.Viewer) | Receives copy, no action |
| Recipient (InPersonSigner) | DocumentSigner (SignerType.InPersonSigner) | Signs in person |
| Tab | FormField | Fields placed on documents |
| SignHere tab | FormField (FieldType.Signature) | Signature placement |
| InitialHere tab | FormField (FieldType.Initial) | Initials placement |
| DateSigned tab | FormField (FieldType.DateSigned) | Auto-filled date |
| Text tab | FormField (FieldType.Textbox) | Free text input |
| Checkbox tab | FormField (FieldType.Checkbox) | Checkbox field |
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

### DocuSign: OAuth 2.0 (JWT or Authorization Code Grant)

```csharp
// DocuSign — JWT or Auth Code flow required
// Every API call needs: accessToken + accountId + basePath
using DocuSign.eSign.Api;
using DocuSign.eSign.Client;

var apiClient = new ApiClient("https://demo.docusign.net/restapi");
apiClient.Configuration.DefaultHeader.Add("Authorization", "Bearer " + accessToken);
var envelopesApi = new EnvelopesApi(apiClient);

// accountId is required in EVERY API call
var result = envelopesApi.CreateEnvelope(accountId, envelopeDefinition);
```

### BoldSign: API Key (simplest) or OAuth 2.0

```csharp
// BoldSign — API Key auth (simplest for server-to-server)
using BoldSign.Api;
using BoldSign.Model;

var apiClient = new ApiClient("https://api.boldsign.com", "YOUR_API_KEY");
var documentClient = new DocumentClient(apiClient);

// No accountId needed — inferred from API key
var result = documentClient.SendDocument(sendForSign);
```

```csharp
// BoldSign — OAuth 2.0 (for user-delegated access)
var configuration = new Configuration();
configuration.SetBearerToken("your-oauth2-access-token");
var apiClient = new ApiClient(configuration);
var documentClient = new DocumentClient(apiClient);
```

### Key Authentication Differences

| Aspect | DocuSign | BoldSign |
|---|---|---|
| Simplest auth | OAuth JWT (still complex) | API Key (single header) |
| Auth header | `Authorization: Bearer {token}` | `X-API-KEY: {key}` or `Authorization: Bearer {token}` |
| Account ID | Required in every API call | Not required (inferred from key/token) |
| Base URL (dev) | `https://demo.docusign.net/restapi` | `https://api.boldsign.com` (sandbox toggle in dashboard) |
| Base URL (prod) | `https://na1.docusign.net/restapi` (varies by region) | `https://api.boldsign.com` (US) / `https://eu-api.boldsign.com` (EU) |
| SDK package | `DocuSign.eSign` (NuGet) | `BoldSign.Api` (NuGet) |
| Token refresh | Developer manages refresh token rotation | API Key never expires; OAuth uses sliding refresh |

### Migration Action

1. Replace `Install-Package DocuSign.eSign` with `Install-Package BoldSign.Api`
2. Replace OAuth token management with a simple API key header (for server-to-server)
3. Remove all `accountId` parameters from API calls
4. Update base URL from DocuSign endpoint to BoldSign endpoint

---

## SDK Installation Migration

### DocuSign

```bash
# DocuSign
dotnet add package DocuSign.eSign
# or
Install-Package DocuSign.eSign
```

### BoldSign

```bash
# BoldSign
dotnet add package BoldSign.Api
# or
Install-Package BoldSign.Api
```

**NuGet:** https://www.nuget.org/packages/BoldSign.Api
**SDK Docs:** https://developers.boldsign.com/sdks/dotnet-sdk/

---

## Send Document Migration

This is the most common operation. DocuSign calls it "creating an envelope"; BoldSign calls it "sending a document."

### DocuSign: Create and Send Envelope

```csharp
using DocuSign.eSign.Api;
using DocuSign.eSign.Client;
using DocuSign.eSign.Model;

// 1. Setup client
var apiClient = new ApiClient("https://demo.docusign.net/restapi");
apiClient.Configuration.DefaultHeader.Add("Authorization", "Bearer " + accessToken);
var envelopesApi = new EnvelopesApi(apiClient);

// 2. Create document
var doc = new Document
{
    DocumentBase64 = Convert.ToBase64String(File.ReadAllBytes("contract.pdf")),
    Name = "Contract.pdf",
    FileExtension = "pdf",
    DocumentId = "1"
};

// 3. Create signer with tabs
var signer = new Signer
{
    Email = "john@example.com",
    Name = "John Doe",
    RecipientId = "1",
    RoutingOrder = "1",
    ClientUserId = "1234",  // For embedded signing
    Tabs = new Tabs
    {
        SignHereTabs = new List<SignHere>
        {
            new SignHere
            {
                DocumentId = "1",
                PageNumber = "1",
                XPosition = "100",
                YPosition = "200"
            }
        },
        DateSignedTabs = new List<DateSigned>
        {
            new DateSigned
            {
                DocumentId = "1",
                PageNumber = "1",
                XPosition = "300",
                YPosition = "200"
            }
        },
        TextTabs = new List<Text>
        {
            new Text
            {
                DocumentId = "1",
                PageNumber = "1",
                XPosition = "100",
                YPosition = "300",
                TabLabel = "company_name",
                Required = "true"
            }
        }
    }
};

// 4. Add CC recipient
var cc = new CarbonCopy
{
    Email = "manager@example.com",
    Name = "Manager",
    RecipientId = "2",
    RoutingOrder = "2"
};

// 5. Build and send envelope
var envelopeDefinition = new EnvelopeDefinition
{
    EmailSubject = "Please sign this document",
    EmailBlurb = "Please review and sign.",
    Documents = new List<Document> { doc },
    Recipients = new Recipients
    {
        Signers = new List<Signer> { signer },
        CarbonCopies = new List<CarbonCopy> { cc }
    },
    Status = "sent"  // "sent" sends immediately; "created" saves as draft
};

var result = envelopesApi.CreateEnvelope(accountId, envelopeDefinition);
Console.WriteLine($"Envelope ID: {result.EnvelopeId}");
```

### BoldSign: Send Document (Equivalent)

```csharp
using BoldSign.Api;
using BoldSign.Model;

// 1. Setup client — much simpler
var apiClient = new ApiClient("https://api.boldsign.com",
    Environment.GetEnvironmentVariable("BOLDSIGN_API_KEY"));
var documentClient = new DocumentClient(apiClient);

// 2. Load file as stream (not Base64)
using var fileStream = File.OpenRead("contract.pdf");
var documentFile = new DocumentFileStream
{
    ContentType = "application/pdf",
    FileData = fileStream,
    FileName = "contract.pdf"
};

// 3. Create form fields (equivalent to DocuSign tabs)
var signatureField = new FormField
{
    FieldType = FieldType.Signature,
    PageNumber = 1,
    Bounds = new Rectangle(x: 100, y: 200, width: 200, height: 50),
    IsRequired = true
};

var dateSignedField = new FormField
{
    FieldType = FieldType.DateSigned,
    PageNumber = 1,
    Bounds = new Rectangle(x: 300, y: 200, width: 150, height: 30)
};

var textField = new FormField
{
    Id = "company_name",
    FieldType = FieldType.Textbox,
    PageNumber = 1,
    Bounds = new Rectangle(x: 100, y: 300, width: 200, height: 30),
    IsRequired = true
};

// 4. Create signer (with form fields attached to the signer)
var signer = new DocumentSigner(
    name: "John Doe",
    emailAddress: "john@example.com",
    signerType: SignerType.Signer,
    formFields: new List<FormField> { signatureField, dateSignedField, textField }
);

// 5. Add viewer (equivalent to DocuSign CC recipient)
var viewer = new DocumentSigner(
    name: "Manager",
    emailAddress: "manager@example.com",
    signerType: SignerType.Viewer
);

// 6. Build and send
var sendRequest = new SendForSign
{
    Title = "Please sign this document",
    Message = "Please review and sign.",
    Signers = new List<DocumentSigner> { signer, viewer },
    Files = new List<IDocumentFile> { documentFile },
    // ExpiryDays = 30,
    // EnableSigningOrder = true  // For sequential signing
};

var result = documentClient.SendDocument(sendRequest);
Console.WriteLine($"Document ID: {result.DocumentId}");
// NOTE: Document send is ASYNC — listen for Sent/SendFailed webhooks
```

### Key Differences When Sending

| Aspect | DocuSign | BoldSign |
|---|---|---|
| File format | Base64 string in JSON | File stream (multipart/form-data) or FileUrl |
| Tabs/Fields | Attached to recipient's `Tabs` property | Attached to signer's `FormFields` list |
| Tab positioning | `XPosition`/`YPosition` (string) | `Bounds` rectangle (x, y, width, height) |
| CC recipients | Separate `CarbonCopy` recipient type | `DocumentSigner` with `SignerType.Viewer` |
| Send status | `Status = "sent"` to send immediately | Sends immediately on API call (no draft by default) |
| Draft mode | `Status = "created"` | Use embedded request for draft-like behavior |
| Document ID | String in response (`EnvelopeId`) | String in response (`DocumentId`) |
| Async behavior | Synchronous (envelope created on return) | **Asynchronous** — must listen for `Sent`/`SendFailed` webhook |
| Subject/Message | `EmailSubject` / `EmailBlurb` | `Title` / `Message` |

---

## Tab → Form Field Mapping (Detailed)

### DocuSign Tab Types → BoldSign FieldType

| DocuSign Tab | DocuSign C# Class | BoldSign FieldType | BoldSign Notes |
|---|---|---|---|
| `signHereTabs` | `SignHere` | `FieldType.Signature` | Direct equivalent |
| `initialHereTabs` | `InitialHere` | `FieldType.Initial` | Direct equivalent |
| `dateSignedTabs` | `DateSigned` | `FieldType.DateSigned` | Auto-populated with sign date |
| `textTabs` | `Text` | `FieldType.Textbox` | Free-text input |
| `checkboxTabs` | `Checkbox` | `FieldType.Checkbox` | Direct equivalent |
| `radioGroupTabs` | `RadioGroup` | `FieldType.RadioButton` | Direct equivalent |
| `listTabs` | `List` | `FieldType.Dropdown` | Dropdown selection |
| `noteTabs` | `Note` | `FieldType.Label` | Read-only display text |
| `attachmentTabs` | `SignerAttachment` | `FieldType.Attachment` | Signer uploads file |
| `emailTabs` | `Email` | `FieldType.Textbox` | Use Textbox with validation |
| `fullNameTabs` | `FullName` | `FieldType.Label` | Pre-fill with signer's name |
| `companyTabs` | `Company` | `FieldType.Textbox` | Text field |
| `titleTabs` | `Title` | `FieldType.Textbox` | Text field |
| `numberTabs` | `ModelNumber` | `FieldType.Textbox` | Textbox (validate in app) |
| `dateTabs` | `ModelDate` | `FieldType.EditableDate` | Date picker |
| `formulaTabs` | `FormulaTab` | *(Custom logic)* | Use Label with pre-calculated value |
| `approveTabs` | `Approve` | *(Not supported)* | Handle in app logic |
| `declineTabs` | `Decline` | *(Built-in)* | Signers can decline natively |
| `drawTabs` | `Draw` | `FieldType.Image` | Image upload as alternative |
| `envelopeIdTabs` | `EnvelopeId` | `FieldType.Label` | Pre-fill with DocumentId |

### Positioning Conversion

```csharp
// DocuSign: separate X/Y position strings, tab on a specific document
var signHere = new SignHere
{
    DocumentId = "1",
    PageNumber = "1",
    XPosition = "100",
    YPosition = "200"
};

// BoldSign: Rectangle bounds with x, y, width, height (all integers)
var signatureField = new FormField
{
    FieldType = FieldType.Signature,
    PageNumber = 1,   // int, not string
    Bounds = new Rectangle(x: 100, y: 200, width: 200, height: 50),
    IsRequired = true
};
```

### Anchor Tags / AutoPlace → Text Tags

```csharp
// DocuSign: Anchor string for auto-placement
var signHere = new SignHere
{
    AnchorString = "/sig1/",
    AnchorXOffset = "20",
    AnchorYOffset = "10",
    AnchorUnits = "pixels"
};

// BoldSign: Text Tags (embed in document before upload)
// In your Word/PDF document, add invisible text:
//   {{:sign:signer1}}       → Signature field
//   {{:initial:signer1}}    → Initials field
//   {{:date:signer1}}       → Date signed
//   {{:textbox:signer1}}    → Text input
// BoldSign auto-converts these to real fields on upload.
// Reference: https://developers.boldsign.com/documents/text-tags/
```

---

## Recipient Type Migration

### DocuSign Recipient Types → BoldSign Signer Types

```csharp
// DocuSign: Multiple recipient types in separate lists
var recipients = new Recipients
{
    Signers = new List<Signer>
    {
        new Signer { Email = "signer@example.com", Name = "Signer",
                     RecipientId = "1", RoutingOrder = "1" }
    },
    CarbonCopies = new List<CarbonCopy>
    {
        new CarbonCopy { Email = "cc@example.com", Name = "CC Person",
                         RecipientId = "2", RoutingOrder = "2" }
    },
    InPersonSigners = new List<InPersonSigner>
    {
        new InPersonSigner { HostEmail = "host@example.com", HostName = "Host",
                             SignerName = "In-Person Signer",
                             RecipientId = "3", RoutingOrder = "3" }
    }
};
```

```csharp
// BoldSign: Unified signer list with SignerType enum
var signers = new List<DocumentSigner>
{
    new DocumentSigner(
        name: "Signer",
        emailAddress: "signer@example.com",
        signerType: SignerType.Signer,
        signerOrder: 1,
        formFields: new List<FormField> { signatureField }
    ),
    new DocumentSigner(
        name: "CC Person",
        emailAddress: "cc@example.com",
        signerType: SignerType.Viewer,  // CC equivalent
        signerOrder: 2
    ),
    new DocumentSigner(
        name: "In-Person Signer",
        emailAddress: "inperson@example.com",
        signerType: SignerType.InPersonSigner,
        signerOrder: 3,
        formFields: new List<FormField> { signatureField2 }
    )
};
```

| DocuSign Recipient | BoldSign SignerType | Notes |
|---|---|---|
| `Signer` | `SignerType.Signer` | Direct equivalent |
| `CarbonCopy` | `SignerType.Viewer` | Receives copy, no action |
| `InPersonSigner` | `SignerType.InPersonSigner` | Direct equivalent |
| `CertifiedDelivery` | `SignerType.Viewer` | Use Viewer (closest equivalent) |
| `Editor` | *(Not supported)* | Not available in BoldSign |
| `Agent` | *(Not supported)* | Not available in BoldSign |
| `Intermediary` | *(Not supported)* | Not available in BoldSign |
| `Witness` | *(Not supported)* | Not available in BoldSign |

---

## Sequential / Parallel Signing Migration

### DocuSign: Routing Order

```csharp
// DocuSign: routingOrder determines signing sequence
var signer1 = new Signer { RoutingOrder = "1", /* ... */ };
var signer2 = new Signer { RoutingOrder = "2", /* ... */ };
// DocuSign always enforces routing order when set
```

### BoldSign: Signer Order + EnableSigningOrder

```csharp
// BoldSign: Must explicitly enable signing order
var signer1 = new DocumentSigner(
    name: "First Signer",
    emailAddress: "first@example.com",
    signerType: SignerType.Signer,
    signerOrder: 1,
    formFields: new List<FormField> { field1 }
);

var signer2 = new DocumentSigner(
    name: "Second Signer",
    emailAddress: "second@example.com",
    signerType: SignerType.Signer,
    signerOrder: 2,
    formFields: new List<FormField> { field2 }
);

var sendRequest = new SendForSign
{
    Title = "Sequential Contract",
    Signers = new List<DocumentSigner> { signer1, signer2 },
    Files = new List<IDocumentFile> { documentFile },
    EnableSigningOrder = true  // REQUIRED for sequential signing
    // Set to false (default) for parallel signing
};
```

---

## Embedded Signing Migration

### DocuSign: CreateRecipientView

```csharp
// DocuSign: Two-step process
// Step 1: Create envelope with clientUserId on signer
var signer = new Signer
{
    Email = "signer@example.com",
    Name = "John Doe",
    RecipientId = "1",
    RoutingOrder = "1",
    ClientUserId = "1234"  // Marks as embedded signer
};

var envelopeDefinition = new EnvelopeDefinition
{
    /* ... documents, recipients ... */
    Status = "sent"
};

var envelopesApi = new EnvelopesApi(apiClient);
var result = envelopesApi.CreateEnvelope(accountId, envelopeDefinition);
string envelopeId = result.EnvelopeId;

// Step 2: Generate signing URL
var recipientViewRequest = new RecipientViewRequest
{
    AuthenticationMethod = "none",
    ClientUserId = "1234",    // Must match
    UserName = "John Doe",    // Must match
    Email = "signer@example.com", // Must match
    ReturnUrl = "https://yourapp.com/signing-complete"
};

ViewUrl viewUrl = envelopesApi.CreateRecipientView(accountId, envelopeId, recipientViewRequest);
string signingUrl = viewUrl.Url;  // Expires in 5 minutes
```

### BoldSign: GetEmbeddedSignLink

```csharp
// BoldSign: Also two-step, but simpler
// Step 1: Send document normally
var signer = new DocumentSigner(
    name: "John Doe",
    emailAddress: "signer@example.com",
    signerType: SignerType.Signer,
    formFields: new List<FormField> { signatureField }
);

var sendRequest = new SendForSign
{
    Title = "Contract for Signing",
    Signers = new List<DocumentSigner> { signer },
    Files = new List<IDocumentFile> { documentFile }
};

var documentClient = new DocumentClient(apiClient);
var result = documentClient.SendDocument(sendRequest);
string documentId = result.DocumentId;

// Step 2: Generate embedded signing URL
var signLinkExpiry = DateTime.UtcNow.AddHours(24);
var signLinkResult = documentClient.GetEmbeddedSignLink(
    documentId: documentId,
    signerEmail: "signer@example.com",
    signLinkValidTill: signLinkExpiry,
    redirectUrl: "https://yourapp.com/signing-complete"
);

string signingUrl = signLinkResult.SignLink;
// Embed in <iframe> or redirect user to this URL
```

### Key Embedded Signing Differences

| Aspect | DocuSign | BoldSign |
|---|---|---|
| Embedded marker | `ClientUserId` on signer (required) | No special marker needed |
| URL generation | `CreateRecipientView` with matching name/email/clientUserId | `GetEmbeddedSignLink` with documentId + signerEmail |
| URL expiry | 5 minutes (default) | Configurable via `SignLinkValidTill` |
| Match requirements | userName + email + clientUserId must all match exactly | signerEmail must match |
| Redirect | `ReturnUrl` in RecipientViewRequest | `RedirectUrl` parameter |
| iFrame support | Supported (with FrameAncestors) | Supported natively |

---

## Embedded Send (Sender View) Migration

### DocuSign: CreateSenderView

```csharp
// DocuSign: Create draft envelope, then get sender view
envelopeDefinition.Status = "created";  // Draft
var result = envelopesApi.CreateEnvelope(accountId, envelopeDefinition);

var senderViewRequest = new ReturnUrlRequest
{
    ReturnUrl = "https://yourapp.com/send-complete"
};

ViewUrl senderView = envelopesApi.CreateSenderView(accountId, result.EnvelopeId, senderViewRequest);
string senderUrl = senderView.Url;
```

### BoldSign: CreateEmbeddedRequestUrl

```csharp
// BoldSign: Create embedded send request
var signer = new DocumentSigner(
    name: "Signer",
    emailAddress: "signer@example.com",
    signerType: SignerType.Signer
);

var embeddedRequest = new EmbeddedDocumentRequest
{
    Title = "Contract",
    Signers = new List<DocumentSigner> { signer },
    Files = new List<IDocumentFile> { documentFile },
    ShowToolbar = true,
    ShowSaveButton = true,
    ShowSendButton = true,
    ShowNavigationButtons = true,
    RedirectUrl = "https://yourapp.com/send-complete"
};

var result = documentClient.CreateEmbeddedRequestUrl(embeddedRequest);
string senderUrl = result.SendUrl;
// Embed in <iframe>
```

---

## Template Migration

### DocuSign: Create Envelope from Template

```csharp
// DocuSign: Use template with role-based recipients
var templateRole = new TemplateRole
{
    Email = "signer@example.com",
    Name = "John Doe",
    RoleName = "Signer"  // Must match template role name
};

var envelopeDefinition = new EnvelopeDefinition
{
    TemplateId = "DOCUSIGN_TEMPLATE_ID",
    TemplateRoles = new List<TemplateRole> { templateRole },
    Status = "sent"
};

var result = envelopesApi.CreateEnvelope(accountId, envelopeDefinition);
```

### BoldSign: Send from Template

```csharp
// BoldSign: Use template with role-indexed recipients
var templateClient = new TemplateClient(apiClient);

var role = new Roles
{
    RoleIndex = 1,   // Index-based (1, 2, 3...), not name-based
    SignerName = "John Doe",
    SignerEmail = "signer@example.com"
};

var sendRequest = new SendForSignFromTemplate
{
    TemplateId = "BOLDSIGN_TEMPLATE_ID",
    Title = "Contract from Template",
    Roles = new List<Roles> { role }
};

var result = templateClient.SendUsingTemplate(sendRequest);
Console.WriteLine($"Document ID: {result.DocumentId}");
```

### Key Template Differences

| Aspect | DocuSign | BoldSign |
|---|---|---|
| Role mapping | By `RoleName` (string match) | By `RoleIndex` (integer, 1-based) |
| Template creation | Synchronous | **Asynchronous** — listen for `TemplateCreated` webhook |
| SDK class | `EnvelopesApi` with template parameters | Dedicated `TemplateClient` |
| Prefill fields | Via `TemplateRole.Tabs` | Via `Roles.FormFields` or prefill API |
| Bulk send | `BulkEnvelopes` endpoint | `POST /v1/template/bulkSend` |

---

## Webhook Migration (DocuSign Connect → BoldSign Webhooks)

### DocuSign: Connect Configuration or EventNotification

```csharp
// DocuSign Option 1: Per-envelope webhook (eventNotification)
var eventNotification = new EventNotification
{
    Url = "https://yourapp.com/webhooks/docusign",
    RequireAcknowledgment = "true",
    LoggingEnabled = "true",
    EnvelopeEvents = new List<EnvelopeEvent>
    {
        new EnvelopeEvent { EnvelopeEventStatusCode = "completed" },
        new EnvelopeEvent { EnvelopeEventStatusCode = "declined" },
        new EnvelopeEvent { EnvelopeEventStatusCode = "voided" }
    },
    RecipientEvents = new List<RecipientEvent>
    {
        new RecipientEvent { RecipientEventStatusCode = "Completed" },
        new RecipientEvent { RecipientEventStatusCode = "Declined" },
        new RecipientEvent { RecipientEventStatusCode = "AuthenticationFailed" }
    }
};

envelopeDefinition.EventNotification = eventNotification;

// DocuSign Option 2: Account-level Connect configuration
var connectApi = new ConnectApi(apiClient);
var connectConfig = new ConnectCustomConfiguration
{
    Name = "My Connect Config",
    UrlToPublishTo = "https://yourapp.com/webhooks/docusign",
    AllUsers = "true",
    EnvelopeEvents = new List<string> { "Completed", "Declined", "Voided" }
};
connectApi.CreateConfiguration(accountId, connectConfig);
```

### BoldSign: Webhook Configuration

```
// BoldSign: Configure webhooks in the BoldSign Web App
// BoldSign App → Settings → Webhooks → Add webhook URL
//
// OR use the API to configure programmatically:
// POST /v1/webhook/create
//
// Two types:
// - Account-level: fires for all documents (web app or API)
// - App-level: fires only for a specific OAuth app
```

### Webhook Event Mapping

| DocuSign Event | BoldSign Event | Notes |
|---|---|---|
| Envelope `Sent` | `Sent` | Document successfully sent |
| Envelope `Delivered` | `Viewed` | Recipient opened document |
| Envelope `Completed` | `Completed` | All signers finished |
| Envelope `Declined` | `Declined` | Signer declined |
| Envelope `Voided` | `Revoked` | Sender voided/revoked |
| *(No equivalent)* | `SendFailed` | **Important** — handle async send failures |
| Recipient `Completed` | `Signed` | Individual signer completed |
| Recipient `Sent` | `Sent` | Document sent to signer |
| Recipient `Delivered` | `Viewed` | Signer opened document |
| Recipient `Declined` | `Declined` | Signer declined |
| Recipient `AuthenticationFailed` | `IdentityVerificationFailed` | Auth failed |
| *(No equivalent)* | `Expired` | Document expired |
| *(No equivalent)* | `TemplateCreated` / `TemplateCreateFailed` | Template async result |
| *(No equivalent)* | `SenderIdentityApproved` / `Denied` / `Revoked` | On-behalf-of lifecycle |

### Webhook Handler Migration

```csharp
// DocuSign: Connect webhook handler (XML or JSON)
[HttpPost("webhooks/docusign")]
public IActionResult HandleDocuSign()
{
    // DocuSign sends XML by default (JSON optional with Connect v2.1)
    // Verification via HMAC or basic auth
    var body = new StreamReader(Request.Body).ReadToEnd();
    var payload = JsonSerializer.Deserialize<JsonElement>(body);
    var status = payload.GetProperty("status").GetString();
    var envelopeId = payload.GetProperty("envelopeId").GetString();
    // ...
}
```

```csharp
// BoldSign: Webhook handler (always JSON)
[ApiController]
[Route("webhooks")]
public class BoldSignWebhookController : ControllerBase
{
    [HttpPost("boldsign")]
    public async Task<IActionResult> HandleWebhook()
    {
        // 1. Read raw body
        using var reader = new StreamReader(Request.Body);
        var body = await reader.ReadToEndAsync();

        // 2. Verify HMAC-SHA256 signature (REQUIRED for security)
        var signature = Request.Headers["X-BoldSign-Signature"].ToString();
        var secret = Environment.GetEnvironmentVariable("BOLDSIGN_WEBHOOK_SECRET");
        var keyBytes = Encoding.UTF8.GetBytes(secret);
        var messageBytes = Encoding.UTF8.GetBytes(body);
        using var hmac = new HMACSHA256(keyBytes);
        var hash = hmac.ComputeHash(messageBytes);
        var expectedSig = Convert.ToHexString(hash).ToLower();

        if (signature != expectedSig)
            return Unauthorized();

        // 3. Parse and handle events
        var payload = JsonSerializer.Deserialize<JsonElement>(body);
        var eventType = payload.GetProperty("event").GetProperty("eventType").GetString();
        var documentId = payload.GetProperty("document").GetProperty("documentId").GetString();

        switch (eventType)
        {
            case "Completed":
                await HandleCompleted(documentId);
                break;
            case "SendFailed":
                // IMPORTANT: Handle async send failures
                Console.WriteLine($"Send failed for {documentId}");
                break;
            case "Signed":
                // Individual signer completed
                break;
            case "Declined":
                Console.WriteLine($"Declined: {documentId}");
                break;
            case "Expired":
                Console.WriteLine($"Expired: {documentId}");
                break;
        }

        return Ok();
    }

    private async Task HandleCompleted(string documentId)
    {
        var apiClient = new ApiClient("https://api.boldsign.com",
            Environment.GetEnvironmentVariable("BOLDSIGN_API_KEY"));
        var documentClient = new DocumentClient(apiClient);

        // Download signed PDF
        var pdf = documentClient.DownloadDocument(documentId);
        await File.WriteAllBytesAsync($"signed/{documentId}.pdf", pdf);

        // Download audit trail
        var audit = documentClient.DownloadAuditLog(documentId);
        await File.WriteAllBytesAsync($"signed/{documentId}-audit.pdf", audit);
    }
}
```

---

## Document Status / Properties Migration

### DocuSign: GetEnvelope

```csharp
// DocuSign
var envelope = envelopesApi.GetEnvelope(accountId, envelopeId);
Console.WriteLine($"Status: {envelope.Status}");
// Statuses: created, sent, delivered, completed, declined, voided
```

### BoldSign: GetDocumentProperties

```csharp
// BoldSign
var details = documentClient.GetDocumentProperties(documentId);
Console.WriteLine($"Status: {details.Status}");
// Statuses: WaitingForOthers (InProgress), Completed, Declined, Expired, Revoked
```

### Status Mapping

| DocuSign Status | BoldSign Status | Notes |
|---|---|---|
| `created` | *(No equivalent)* | BoldSign doesn't have persistent drafts |
| `sent` | `WaitingForOthers` | Document sent, awaiting signatures |
| `delivered` | `WaitingForOthers` | Viewed but not yet signed |
| `completed` | `Completed` | All signers finished |
| `declined` | `Declined` | Signer declined |
| `voided` | `Revoked` | Sender cancelled |
| *(No equivalent)* | `Expired` | Document expired |

---

## Download Signed Document Migration

### DocuSign

```csharp
// DocuSign: Download combined document
var documents = envelopesApi.GetDocument(accountId, envelopeId, "combined");
File.WriteAllBytes("signed-combined.pdf", documents);
```

### BoldSign

```csharp
// BoldSign: Download signed document
var pdf = documentClient.DownloadDocument(documentId);
File.WriteAllBytes("signed.pdf", pdf);

// Download audit trail (Certificate of Completion equivalent)
var audit = documentClient.DownloadAuditLog(documentId);
File.WriteAllBytes("audit-trail.pdf", audit);
```

---

## List / Search Documents Migration

### DocuSign: ListStatusChanges

```csharp
// DocuSign: Search envelopes with filters
var options = new EnvelopesApi.ListStatusChangesOptions
{
    fromDate = DateTime.UtcNow.AddDays(-30).ToString("o"),
    status = "completed",
    include = "recipients"
};

var results = envelopesApi.ListStatusChanges(accountId, options);
foreach (var envelope in results.Envelopes)
{
    Console.WriteLine($"{envelope.EnvelopeId}: {envelope.Status}");
}
```

### BoldSign: ListDocuments

```csharp
// BoldSign: List documents with filters
var documents = documentClient.ListDocuments(
    page: 1,
    pageSize: 20
    // Additional filters available via query parameters
);

foreach (var doc in documents.Result)
{
    Console.WriteLine($"{doc.DocumentId}: {doc.Status}");
}
```

---

## Remind / Void / Delete Migration

### DocuSign

```csharp
// Remind
envelopesApi.UpdateRecipients(accountId, envelopeId, /* resend reminder */);

// Void envelope
var voidRequest = new VoidEnvelopeRequest { VoidedReason = "No longer needed" };
envelopesApi.Update(accountId, envelopeId, new Envelope { Status = "voided", VoidedReason = "Cancelled" });

// Delete (purge)
envelopesApi.Update(accountId, envelopeId, new Envelope { PurgeState = "documents_and_metadata_queued" });
```

### BoldSign

```csharp
// Remind specific signer
documentClient.RemindDocument(new ReminderMessage
{
    DocumentId = documentId,
    ReceiverEmails = new List<string> { "signer@example.com" }
});

// Revoke (void) document
documentClient.RevokeDocument(new RevokeDocument
{
    DocumentId = documentId,
    Message = "No longer needed"
});

// Delete document
documentClient.DeleteDocument(documentId);
```

---

## Sender Identity / On-Behalf-Of Migration

### DocuSign: Send on behalf of another user

```csharp
// DocuSign: Requires OAuth with specific user's token
// Each user needs their own OAuth consent or JWT sub claim
// The authenticated user IS the sender
```

### BoldSign: Sender Identity (much simpler)

```csharp
// BoldSign: Register identity once, then use onBehalfOf
var senderIdentityClient = new SenderIdentityClient(apiClient);

// Step 1: Register tenant identity (one-time)
senderIdentityClient.CreateSenderIdentity(new CreateSenderIdentityRequest
{
    Name = "Acme Corp",
    Email = "admin@acme.com"
});

// Step 2: Request approval (BoldSign sends email to tenant)
senderIdentityClient.RequestForIdentityApproval(new ResendSenderIdentityRequest
{
    Email = "admin@acme.com"
});
// Tenant clicks Approve → SenderIdentityApproved webhook fires

// Step 3: Send on behalf of tenant
var sendRequest = new SendForSign
{
    OnBehalfOf = "admin@acme.com",  // ← key field
    Title = "Contract from Acme",
    Signers = new List<DocumentSigner> { signer },
    Files = new List<IDocumentFile> { documentFile }
};

documentClient.SendDocument(sendRequest);
```

---

## Signer Authentication Migration

### DocuSign

```csharp
// DocuSign: Access code on recipient
var signer = new Signer
{
    AccessCode = "12345",  // Signer must enter this code
    /* ... */
};

// Or phone authentication, ID verification, etc.
```

### BoldSign

```csharp
// BoldSign: Authentication options on signer
var signer = new DocumentSigner(
    name: "John Doe",
    emailAddress: "john@example.com",
    signerType: SignerType.Signer,
    authenticationType: AuthenticationType.AccessCode,
    formFields: new List<FormField> { signatureField }
);
// Also supports: EmailOTP, SMSOTP, IdVerification
```

| DocuSign Auth | BoldSign Auth | Notes |
|---|---|---|
| Access Code | `AuthenticationType.AccessCode` | Direct equivalent |
| Phone Auth | `AuthenticationType.SMSOTP` | SMS OTP |
| *(No direct equivalent)* | `AuthenticationType.EmailOTP` | Email-based OTP |
| ID Verification | `AuthenticationType.IdVerification` | Passport/Gov ID |
| Knowledge-Based Auth | *(Not supported)* | Not available |

---

## REST API Endpoint Mapping

| Operation | DocuSign Endpoint | BoldSign Endpoint |
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

### DocuSign appsettings.json

```json
{
  "DocuSign": {
    "IntegrationKey": "your-integration-key",
    "SecretKey": "your-secret-key",
    "UserId": "your-user-id-guid",
    "AccountId": "your-account-id",
    "AuthServer": "account-d.docusign.com",
    "BasePath": "https://demo.docusign.net/restapi",
    "PrivateKeyFile": "private.key"
  }
}
```

### BoldSign appsettings.json

```json
{
  "BoldSign": {
    "ApiKey": "your_api_key_here",
    "WebhookSecret": "your_webhook_secret_here",
    "BaseUrl": "https://api.boldsign.com"
  }
}
```

---

## Rate Limit Comparison

| Aspect | DocuSign | BoldSign |
|---|---|---|
| Polling limit | Max once per 15 minutes | Use webhooks instead of polling |
| API burst limit | 10 requests per second (varies by plan) | Sandbox: 50 req/hour; Live: 2,000 req/hour |
| Rate limit scope | Per integration key | Per account (all keys combined) |
| Rate limit headers | `X-RateLimit-*` | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` |
| Exceeded response | 429 Too Many Requests | 429 Too Many Requests + `Retry-After` header |

### Rate Limit Handler

```csharp
// BoldSign rate limit handling
public async Task<T> CallWithRetryAsync<T>(Func<Task<T>> fn, int maxRetries = 1)
{
    for (int attempt = 0; attempt <= maxRetries; attempt++)
    {
        try
        {
            return await fn();
        }
        catch (ApiException ex) when (ex.ErrorCode == 429)
        {
            if (attempt == maxRetries) throw;
            await Task.Delay(TimeSpan.FromSeconds(60));
        }
    }
    throw new Exception("Max retries exceeded");
}
```

---

## Migration Checklist

### Phase 1: Setup
- [ ] Create BoldSign Sandbox account (https://account.boldsign.com/signup?planId=1076)
- [ ] Replace NuGet: `DocuSign.eSign` → `BoldSign.Api`
- [ ] Replace authentication: OAuth JWT → API Key (for server-to-server)
- [ ] Update base URL configuration
- [ ] Remove all `accountId` references from API calls

### Phase 2: Core Document Workflow
- [ ] Migrate send document flow (EnvelopeDefinition → SendForSign)
- [ ] Convert tabs to form fields (see Tab → FormField mapping)
- [ ] Convert recipients to signers (Signer/CC → DocumentSigner with SignerType)
- [ ] Migrate file handling (Base64 → file stream)
- [ ] Implement webhook handler for `Sent`/`SendFailed` events (async send)
- [ ] Migrate embedded signing (CreateRecipientView → GetEmbeddedSignLink)

### Phase 3: Advanced Features
- [ ] Migrate templates (TemplateRole.RoleName → Roles.RoleIndex)
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
- ❌ Using `application/json` for file uploads — BoldSign requires `multipart/form-data` for file uploads
- ❌ Not setting `EnableSigningOrder = true` when sequential signing is needed
- ❌ Trying to use DocuSign `RoleName` for templates — BoldSign uses integer `RoleIndex`
- ❌ Not verifying webhook `X-BoldSign-Signature` HMAC — required for production security
- ❌ Polling for document status — use webhooks (especially important with 50 req/hour sandbox limit)
- ❌ Passing `accountId` to BoldSign SDK calls — it's not needed and will cause errors
- ❌ Assuming template creation is synchronous — listen for `TemplateCreated` webhook
- ❌ Not handling `Expired` events — DocuSign has no equivalent, but BoldSign documents can expire

---

## Reference Links

**BoldSign:**
- .NET SDK: https://developers.boldsign.com/sdks/dotnet-sdk/
- NuGet: https://www.nuget.org/packages/BoldSign.Api
- GitHub: https://github.com/boldsign/boldsign-csharp-sdk
- API Reference: https://developers.boldsign.com/
- Text Tags: https://developers.boldsign.com/documents/text-tags/
- Webhooks: https://developers.boldsign.com/webhooks/
- Sandbox Signup: https://account.boldsign.com/signup?planId=1076

**DocuSign (for reference during migration):**
- REST API Reference: https://developers.docusign.com/docs/esign-rest-api/reference/
- C# SDK: https://github.com/docusign/docusign-esign-csharp-client
- Tabs Reference: https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/tabs/
- Connect Webhooks: https://developers.docusign.com/platform/webhooks/connect/
