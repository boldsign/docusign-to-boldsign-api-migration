---
name: docusign-to-boldsign-migration-java
description: >
  Complete migration guide for moving from Docusign eSignature API to BoldSign eSignature API
  using Java. Use this skill whenever a developer mentions migrating from Docusign to BoldSign,
  replacing Docusign with BoldSign, switching eSignature providers, or needs help converting
  Docusign Java SDK code to BoldSign Java SDK code. Covers: authentication migration, envelope-to-document
  mapping, recipient-to-signer mapping, tab-to-form-field mapping, embedded signing, templates,
  webhooks (Connect to BoldSign webhooks), sender identity, and complete side-by-side code examples.
  Always trigger this skill for Docusign→BoldSign migration tasks — even if the user just asks
  "Migrate Docusign to BoldSign"
  "how do I convert my Docusign code to BoldSign?" or "what's the BoldSign equivalent of EnvelopesApi?"
-

# Docusign → BoldSign Migration Guide (Java)

You are now a Docusign-to-BoldSign migration expert for Java. Your job is to help developers
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
User pastes Docusign Java code and wants it rewritten for BoldSign
User references docusign-esign-java dependency and wants BoldSign replacement
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
User needs help with BoldSign .NET/C#, Node.js, Python, or PHP migration (those are language-specific skills)

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
| Recipient (Signer) | DocumentSigner (signerType: Signer) | Person who signs |
| Recipient (CC) | DocumentCC | Receives copy, no action |
| Recipient (InPersonSigner) | DocumentSigner (signerType: InPersonSigner) | Signs in person |
| Tab | FormField | Fields placed on documents |
| SignHere tab | FormField (fieldType: Signature) | Signature placement |
| InitialHere tab | FormField (fieldType: Initial) | Initials placement |
| DateSigned tab | FormField (fieldType: DateSigned) | Auto-filled date |
| Text tab | FormField (fieldType: TEXT_BOX) | Free text input |
| Checkbox tab | FormField (fieldType: CHECK_BOX) | Checkbox field |
| RadioGroup tab | FormField (fieldType: RadioButton) | Radio button selection |
| List/Dropdown tab | FormField (fieldType: Dropdown) | Dropdown selection |
| Note tab | FormField (fieldType: Label) | Read-only label |
| Attachment tab | FormField (fieldType: Attachment) | File attachment |
| Formula tab | *(Not directly supported)* | Use Label with prefilled value |
| Approve tab | *(Not directly supported)* | Use custom workflow |
| Decline tab | *(Built-in behavior)* | Signers can decline by default |
| Document (within envelope) | File (within document send) | The actual PDF/file |
| RecipientViewRequest | getEmbeddedSignLink | Embedded signing URL |
| EventNotification | Webhook configuration | Webhook setup |
| Connect (account webhook) | Account-level webhook | BoldSign App → Settings → Webhooks |
| Anchor string / AutoPlace | Text Tags | Auto-place fields using text markers |
| Routing order | SignerOrder + EnableSigningOrder | Sequential signing |
| Custom fields (envelope) | *(Use metadata/labels)* | Envelope-level metadata |
| Brands | BrandId | Custom branding per document |
| PowerForms | *(Not directly supported)* | Use embedded request + template |
| accountId (required in every call) | *(Not required)* | BoldSign infers from API key/token |

> **Note:** SDK package names referenced throughout this guide use `boldsign-java` (not `boldsign-java-sdk`)

---

## Authentication Migration

### Docusign: OAuth 2.0 (JWT or Authorization Code Grant)

```java
import com.docusign.esign.client.ApiClient;
import com.docusign.esign.client.auth.OAuth;
import com.docusign.esign.api.EnvelopesApi;

// Docusign — OAuth required, accountId needed for every call
ApiClient apiClient = new ApiClient("https://demo.docusign.net/restapi");
apiClient.addDefaultHeader("Authorization", "Bearer " + accessToken);

EnvelopesApi envelopesApi = new EnvelopesApi(apiClient);

// accountId is required in EVERY API call
EnvelopeSummary result = envelopesApi.createEnvelope(accountId, envelopeDefinition);
```

### BoldSign: API Key (simplest) or OAuth 2.0

```java
import com.boldsign.ApiClient;
import com.boldsign.api.DocumentApi;
import com.boldsign.model.*;

// BoldSign — API Key auth (simplest for server-to-server)
ApiClient client = Configuration.getDefaultApiClient();  
client.setApiKey("Your-API-Key-Here");

DocumentApi documentApi = new DocumentApi(client);

// No accountId needed — inferred from API key
DocumentCreated result = documentApi.sendDocument(sendForSign);
```

```java
// BoldSign — OAuth 2.0 (for user-delegated access)
ApiClient client = Configuration.getDefaultApiClient();
client.setAccessToken("Your-Bearer-Token-Here");

DocumentApi documentApi = new DocumentApi(client);
```

### Key Authentication Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Simplest auth | OAuth JWT (complex) | API Key (single config) |
| Auth configuration | `Authorization: Bearer {token}` | `X-API-KEY: {key}` or `Bearer {token}` |
| Account ID | Required in every API call | Not required (inferred) |
| Base URL (dev) | `https://demo.docusign.net/restapi` | `https://api.boldsign.com` (sandbox toggle) |
| SDK package | `docusign-esign-java` | `com.boldsign:boldsign-java` |

---

## SDK Installation Migration

### Docusign

**Maven:**
```xml
<dependency>
    <groupId>com.docusign</groupId>
    <artifactId>docusign-esign-java</artifactId>
    <version>3.x.x</version>
</dependency>
```

**Gradle:**
```groovy
implementation 'com.docusign:docusign-esign-java:3.x.x'
```

### BoldSign

**Maven:**
```xml
<dependency>
    <groupId>com.boldsign</groupId>
    <artifactId>boldsign-java</artifactId>
    <version>1.x.x</version>
</dependency>
```

**Gradle:**
```groovy
implementation 'com.boldsign:boldsign-java:1.x.x'
```

**Maven Central:** https://central.sonatype.com/artifact/com.boldsign/boldsign-java  
**SDK Docs:** https://developers.boldsign.com/sdks/java-sdk/

> **Package Name:** The BoldSign SDK for Java uses `com.boldsign:boldsign-java` (not `boldsign-java-sdk`) in Maven coordinates.

---

## Send Document Migration

### Docusign: Create and Send Envelope

```java
import com.docusign.esign.api.EnvelopesApi;
import com.docusign.esign.client.ApiClient;
import com.docusign.esign.model.*;
import java.io.*;
import java.nio.file.*;
import java.util.*;

// Setup
ApiClient apiClient = new ApiClient("https://demo.docusign.net/restapi");
apiClient.addDefaultHeader("Authorization", "Bearer " + accessToken);
EnvelopesApi envelopesApi = new EnvelopesApi(apiClient);

// Create document
byte[] fileBytes = Files.readAllBytes(Paths.get("contract.pdf"));
String base64 = Base64.getEncoder().encodeToString(fileBytes);

Document doc = new Document();
doc.setDocumentBase64(base64);
doc.setName("Contract.pdf");
doc.setFileExtension("pdf");
doc.setDocumentId("1");

// Create signer with tabs
SignHere signHere = new SignHere();
signHere.setDocumentId("1");
signHere.setPageNumber("1");
signHere.setXPosition("100");
signHere.setYPosition("200");

DateSigned dateSigned = new DateSigned();
dateSigned.setDocumentId("1");
dateSigned.setPageNumber("1");
dateSigned.setXPosition("300");
dateSigned.setYPosition("200");

Text textTab = new Text();
textTab.setDocumentId("1");
textTab.setPageNumber("1");
textTab.setXPosition("100");
textTab.setYPosition("300");
textTab.setTabLabel("company_name");
textTab.setRequired("true");

Tabs tabs = new Tabs();
tabs.setSignHereTabs(Arrays.asList(signHere));
tabs.setDateSignedTabs(Arrays.asList(dateSigned));
tabs.setTextTabs(Arrays.asList(textTab));

Signer signer = new Signer();
signer.setEmail("john@example.com");
signer.setName("John Doe");
signer.setRecipientId("1");
signer.setRoutingOrder("1");
signer.setTabs(tabs);

// Add CC recipient
CarbonCopy cc = new CarbonCopy();
cc.setEmail("manager@example.com");
cc.setName("Manager");
cc.setRecipientId("2");
cc.setRoutingOrder("2");

Recipients recipients = new Recipients();
recipients.setSigners(Arrays.asList(signer));
recipients.setCarbonCopies(Arrays.asList(cc));

// Build and send envelope
EnvelopeDefinition envelopeDefinition = new EnvelopeDefinition();
envelopeDefinition.setEmailSubject("Please sign this document");
envelopeDefinition.setEmailBlurb("Please review and sign.");
envelopeDefinition.setDocuments(Arrays.asList(doc));
envelopeDefinition.setRecipients(recipients);
envelopeDefinition.setStatus("sent");  // "sent" sends immediately

EnvelopeSummary result = envelopesApi.createEnvelope(accountId, envelopeDefinition);
System.out.println("Envelope ID: " + result.getEnvelopeId());
```

### BoldSign: Send Document (Equivalent)

```java
import com.boldsign.ApiClient;
import com.boldsign.ApiException;
import com.boldsign.Configuration;
import com.boldsign.api.DocumentApi;
import com.boldsign.model.DocumentCC;
import com.boldsign.model.DocumentCreated;
import com.boldsign.model.DocumentSigner;
import com.boldsign.model.FormField;
import com.boldsign.model.Rectangle;
import com.boldsign.model.SendForSign;
import java.io.File;
import java.util.Arrays;

// Setup — much simpler
DocumentApi documentApi = new DocumentApi(apiClient);

// Create form fields (equivalent to Docusign tabs)
Rectangle bounds = new Rectangle();
bounds.setX(100f);
bounds.setY(200f);
bounds.setWidth(200f);
bounds.setHeight(50f);

FormField signatureField = new FormField();
signatureField.setFieldType(FormField.FieldTypeEnum.SIGNATURE);
signatureField.setPageNumber(1);
signatureField.setBounds(bounds);
signatureField.setIsRequired(true);

Rectangle dateBounds = new Rectangle();
dateBounds.setX(300f);
dateBounds.setY(200f);
dateBounds.setWidth(150f);
dateBounds.setHeight(30f);

FormField dateSignedField = new FormField();
dateSignedField.setFieldType(FormField.FieldTypeEnum.DATE_SIGNED);
dateSignedField.setPageNumber(1);
dateSignedField.setBounds(dateBounds);

Rectangle textBounds = new Rectangle();
textBounds.setX(100f);
textBounds.setY(300f);
textBounds.setWidth(200f);
textBounds.setHeight(30f);

FormField textField = new FormField();
textField.setFieldType(FormField.FieldTypeEnum.TEXT_BOX);
textField.setPageNumber(1);
textField.setBounds(textBounds);
textField.setIsRequired(true);

// Create signer (with form fields attached)
DocumentSigner signer = new DocumentSigner();
signer.setName("John Doe");
signer.setEmailAddress("john@example.com");
signer.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer.setFormFields(Arrays.asList(signatureField, dateSignedField, textField));

// Add CC
DocumentCC cc = new DocumentCC();
cc.setEmailAddress("manager@example.com");

// Build and send
File file = new File("contract.pdf");
SendForSign sendRequest = new SendForSign();
sendRequest.setTitle("Please sign this document");
sendRequest.setMessage("Please review and sign.");
sendRequest.setSigners(Arrays.asList(signer));
sendRequest.setCc(Arrays.asList(cc));
sendRequest.setFiles(Arrays.asList(file));

DocumentCreated result = documentApi.sendDocument(sendRequest);
System.out.println("Document ID: " + result.getDocumentId());
// NOTE: Document send is ASYNC — listen for Sent/SendFailed webhooks
```

### Key Differences When Sending

| Aspect | Docusign | BoldSign |
|---|---|---|
| File format | Base64 string in JSON | File object (binary) |
| Tabs/Fields | Attached to recipient's `tabs` property | Attached to signer's `formFields` list |
| Tab positioning | `xPosition`/`yPosition` (strings) | `bounds` object (x, y, width, height) |
| CC recipients | Separate `carbonCopies` list | Separate `DocumentCC` recipient |
| Send status | `status: "sent"` to send immediately | Sends immediately on API call |
| Async behavior | Synchronous (envelope created on return) | **Asynchronous** — must listen for webhooks |
| Subject/Message | `emailSubject` / `emailBlurb` | `title` / `message` |

---

## Tab → Form Field Mapping (Detailed)

### Docusign Tab Types → BoldSign FieldType

| Docusign Tab | BoldSign FieldTypeEnum.| Notes |
|---|---|---|
| `SignHere` | `FieldTypeEnum.SIGNATURE` | Direct equivalent |
| `InitialHere` | `FieldTypeEnum.INITIAL` | Direct equivalent |
| `DateSigned` | `FieldTypeEnum.DATE_SIGNED` | Auto-populated |
| `Text` | `FieldTypeEnum.TEXT_BOX` | Free-text input |
| `Checkbox` | `FieldTypeEnum.CHECK_BOX` | Direct equivalent |
| `RadioGroup` | `FieldTypeEnum.RADIO_BUTTON` | Direct equivalent |
| `List` | `FieldTypeEnum.DROPDOWN` | Dropdown selection |
| `Note` | `FieldTypeEnum.LABEL` | Read-only display |
| `SignerAttachment` | `FieldTypeEnum.ATTACHMENT` | File attachment |
| `ModelDate` | `FieldTypeEnum.EDITABLE_DATE` | Date picker |

### Positioning Conversion

```java
// Docusign: separate X/Y position strings, tab on a specific document
SignHere signHere = new SignHere();
signHere.setDocumentId("1");
signHere.setPageNumber("1");
signHere.setXPosition("100");
signHere.setYPosition("200");

// BoldSign: Rectangle bounds with x, y, width, height (all integers)
Rectangle bounds = new Rectangle();
bounds.setX(100f);
bounds.setY(200f);
bounds.setWidth(200f);
bounds.setHeight(50f);

FormField signatureField = new FormField();
signatureField.setFieldType(FormField.FieldTypeEnum.SIGNATURE);
signatureField.setPageNumber(1);   // int, not string
signatureField.setBounds(bounds);
signatureField.setIsRequired(true);
```

### Anchor Tags / AutoPlace → Text Tags

```java
// Docusign: Anchor string for auto-placement
SignHere signHere = new SignHere();
signHere.setAnchorString("/sig1/");
signHere.setAnchorXOffset("20");
signHere.setAnchorYOffset("10");
signHere.setAnchorUnits("pixels");

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

```java
// Docusign: Multiple recipient types
Signer signer = new Signer();
signer.setEmail("signer@example.com");
signer.setName("Signer");
signer.setRecipientId("1");
signer.setRoutingOrder("1");

CarbonCopy cc = new CarbonCopy();
cc.setEmail("cc@example.com");
cc.setName("CC Person");
cc.setRecipientId("2");
cc.setRoutingOrder("2");

InPersonSigner inPersonSigner = new InPersonSigner();
inPersonSigner.setSignerName("In-Person Signer");
inPersonSigner.setEmail("inperson@example.com");
inPersonSigner.setRecipientId("3");
inPersonSigner.setRoutingOrder("3");

Recipients recipients = new Recipients();
recipients.setSigners(Arrays.asList(signer));
recipients.setCarbonCopies(Arrays.asList(cc));
recipients.setInPersonSigners(Arrays.asList(inPersonSigner));
```

```java
// BoldSign: Unified signer list with SignerType enum
DocumentSigner signer1 = new DocumentSigner();
signer1.setName("Signer");
signer1.setEmailAddress("signer@example.com");
signer1.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer1.setSignerOrder(1);
signer1.setFormFields(Arrays.asList(signatureField));

DocumentSigner reviewer = new DocumentSigner();
reviewer.setName("CC Person");
reviewer.setEmailAddress("cc@example.com");
reviewer.setSignerType(DocumentSigner.SignerTypeEnum.REVIEWER);
reviewer.setSignerOrder(2);

DocumentSigner inPersonSigner = new DocumentSigner();
inPersonSigner.setName("In-Person Signer");
inPersonSigner.setEmailAddress("inperson@example.com");
inPersonSigner.setSignerType(DocumentSigner.SignerTypeEnum.IN_PERSON_SIGNER);
inPersonSigner.setSignerOrder(3);
inPersonSigner.setFormFields(Arrays.asList(signatureField2));

List<DocumentSigner> signers = Arrays.asList(signer1, reviewer, inPersonSigner);
```

| Docusign Recipient | BoldSign SignerType | Notes |
|---|---|---|
| `Signer` | `SignerTypeEnum.SIGNER` | Direct equivalent |
| `CarbonCopy` | Separate `DocumentCC` recipient | Receives copy, no action |
| `InPersonSigner` | `SignerTypeEnum.IN_PERSON_SIGNER` | Direct equivalent |
| `CertifiedDelivery` | `SignerTypeEnum.REVIEWER` | Use Viewer (closest equivalent) |
| `Editor` | *(Not supported)* | Not available in BoldSign |
| `Agent` | *(Not supported)* | Not available in BoldSign |
| `Intermediary` | *(Not supported)* | Not available in BoldSign |
| `Witness` | *(Not supported)* | Not available in BoldSign |

---

## Sequential / Parallel Signing Migration

### Docusign: Routing Order

```java
// Docusign: routingOrder determines signing sequence
Signer signer1 = new Signer();
signer1.setRoutingOrder("1");

Signer signer2 = new Signer();
signer2.setRoutingOrder("2");
// Docusign always enforces routing order when set
```

### BoldSign: Signer Order + EnableSigningOrder

```java
// BoldSign: Must explicitly enable signing order
DocumentSigner signer1 = new DocumentSigner();
signer1.setName("First Signer");
signer1.setEmailAddress("first@example.com");
signer1.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer1.setSignerOrder(1);
signer1.setFormFields(Arrays.asList(field1));

DocumentSigner signer2 = new DocumentSigner();
signer2.setName("Second Signer");
signer2.setEmailAddress("second@example.com");
signer2.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer2.setSignerOrder(2);
signer2.setFormFields(Arrays.asList(field2));

SendForSign sendRequest = new SendForSign();
sendRequest.setTitle("Sequential Contract");
sendRequest.setSigners(Arrays.asList(signer1, signer2));
sendRequest.setFiles(Arrays.asList(new File("contract.pdf")));
sendRequest.setEnableSigningOrder(true);  // REQUIRED for sequential signing
```

---

## Embedded Signing Migration

### Docusign: CreateRecipientView

```java
import com.docusign.esign.model.RecipientViewRequest;
import com.docusign.esign.model.ViewUrl;

// Step 1: Create envelope with clientUserId
Signer signer = new Signer();
signer.setEmail("signer@example.com");
signer.setName("John Doe");
signer.setRecipientId("1");
signer.setRoutingOrder("1");
signer.setClientUserId("1234");  // Marks as embedded signer

EnvelopeDefinition envelopeDefinition = new EnvelopeDefinition();
// ... set documents, recipients ...
envelopeDefinition.setStatus("sent");

EnvelopeSummary result = envelopesApi.createEnvelope(accountId, envelopeDefinition);
String envelopeId = result.getEnvelopeId();

// Step 2: Generate signing URL
RecipientViewRequest recipientViewRequest = new RecipientViewRequest();
recipientViewRequest.setAuthenticationMethod("none");
recipientViewRequest.setClientUserId("1234");
recipientViewRequest.setUserName("John Doe");
recipientViewRequest.setEmail("signer@example.com");
recipientViewRequest.setReturnUrl("https://yourapp.com/signing-complete");

ViewUrl viewUrl = envelopesApi.createRecipientView(accountId, envelopeId, recipientViewRequest);
String signingUrl = viewUrl.getUrl();  // Expires in 5 minutes
```

### BoldSign: getEmbeddedSignLink

```java

// Step 1: Send document normally
DocumentSigner signer = new DocumentSigner();
signer.setName("John Doe");
signer.setEmailAddress("john@example.com");
signer.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer.setFormFields(Arrays.asList(signatureField));

SendForSign sendRequest = new SendForSign();
sendRequest.setTitle("Contract for Signing");
sendRequest.setSigners(Arrays.asList(signer));
sendRequest.setFiles(Arrays.asList(new File("contract.pdf")));

DocumentCreated result = documentApi.sendDocument(sendRequest);
String documentId = result.getDocumentId();

// Step 2: Generate embedded signing URL
EmbeddedSignLinkResponse signLinkResult = documentApi.getEmbeddedSignLink(
    documentId,
    "john@example.com",
    null,   // countryCode (optional, for SMS OTP)
    null,   // sendSMS
    null, // signLinkValidTill
    "https://yourapp.com/signing-complete" // redirectUrl
);

String signingUrl = signLinkResult.getSignLink();
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

```java
// Docusign: Create draft envelope, then get sender view
envelopeDefinition.setStatus("created");  // Draft
EnvelopeSummary result = envelopesApi.createEnvelope(accountId, envelopeDefinition);

ReturnUrlRequest senderViewRequest = new ReturnUrlRequest();
senderViewRequest.setReturnUrl("https://yourapp.com/send-complete");

ViewUrl senderView = envelopesApi.createSenderView(accountId, result.getEnvelopeId(), senderViewRequest);
String senderUrl = senderView.getUrl();
```

### BoldSign: CreateEmbeddedRequestUrl

```java
// BoldSign: Create embedded send request
DocumentSigner signer = new DocumentSigner();
signer.setName("Signer");
signer.setEmailAddress("signer@example.com");
signer.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);

EmbeddedDocumentRequest embeddedRequest = new EmbeddedDocumentRequest();
embeddedRequest.setTitle("Contract");
embeddedRequest.setSigners(Arrays.asList(signer));
embeddedRequest.setFiles(Arrays.asList(new File("contract.pdf")));
embeddedRequest.setShowToolbar(true);
embeddedRequest.setShowSaveButton(true);
embeddedRequest.setShowSendButton(true);
embeddedRequest.setShowNavigationButtons(true);
embeddedRequest.setRedirectUrl("https://yourapp.com/send-complete");

EmbeddedRequestUrlResponse result = documentApi.createEmbeddedRequestUrl(embeddedRequest);
String senderUrl = result.getSendUrl();
// Embed in <iframe>
```

---

## Template Migration

### Docusign: Create Envelope from Template

```java
import com.docusign.esign.model.TemplateRole;

TemplateRole templateRole = new TemplateRole();
templateRole.setEmail("signer@example.com");
templateRole.setName("John Doe");
templateRole.setRoleName("Signer");  // Must match template role name

EnvelopeDefinition envelopeDefinition = new EnvelopeDefinition();
envelopeDefinition.setTemplateId("DOCUSIGN_TEMPLATE_ID");
envelopeDefinition.setTemplateRoles(Arrays.asList(templateRole));
envelopeDefinition.setStatus("sent");

EnvelopeSummary result = envelopesApi.createEnvelope(accountId, envelopeDefinition);
```

### BoldSign: Send from Template

```java
import com.boldsign.api.TemplateApi;
import com.boldsign.model.Role;
import com.boldsign.model.SendForSignFromTemplateForm;

TemplateApi templateApi = new TemplateApi(apiClient);

Role role = new Role();
role.setRoleIndex(1);   // Index-based (1, 2, 3...), not name-based
role.setSignerName("John Doe");
role.setSignerEmail("signer@example.com");
role.setSignerType(Role.SignerTypeEnum.SIGNER);

SendForSignFromTemplateForm sendForSignFromTemplate  = new SendForSignFromTemplateForm();
sendForSignFromTemplate.setRoles(Arrays.asList(role));

DocumentCreated result = templateApi.sendUsingTemplate("YOUR_TEMPLATE_ID", sendForSignFromTemplate);
System.out.println("Document ID: " + result.getDocumentId());
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

```java
import com.docusign.esign.model.*;

EventNotification eventNotification = new EventNotification();
eventNotification.setUrl("https://yourapp.com/webhooks/docusign");
eventNotification.setRequireAcknowledgment("true");
eventNotification.setLoggingEnabled("true");

EnvelopeEvent envelopeEvent1 = new EnvelopeEvent();
envelopeEvent1.setEnvelopeEventStatusCode("completed");

EnvelopeEvent envelopeEvent2 = new EnvelopeEvent();
envelopeEvent2.setEnvelopeEventStatusCode("declined");

eventNotification.setEnvelopeEvents(Arrays.asList(envelopeEvent1, envelopeEvent2));

envelopeDefinition.setEventNotification(eventNotification);
```

### BoldSign: Webhook Handler (Spring Boot)

```java
import org.springframework.web.bind.annotation.*;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

@RestController
@RequestMapping("/webhooks")
public class BoldSignWebhookController {

    @PostMapping("/boldsign")
    public ResponseEntity<String> handleWebhook(
            @RequestHeader("X-BoldSign-Signature") String signature,
            @RequestBody String payload) {
        
        // 1. Verify HMAC signature
        String secret = System.getenv("BOLDSIGN_WEBHOOK_SECRET");
        String expectedSig = computeHmac(payload, secret);
        
        if (!signature.equals(expectedSig)) {
            return ResponseEntity.status(401).body("Invalid signature");
        }
        
        // 2. Parse and handle event
        ObjectMapper mapper = new ObjectMapper();
        JsonNode json = mapper.readTree(payload);
        
        String eventType = json.get("event").get("eventType").asText();
        String documentId = json.get("document").get("documentId").asText();
        
        switch (eventType) {
            case "Completed":
                handleCompleted(documentId);
                break;
            case "SendFailed":
                System.err.println("Send failed for " + documentId);
                break;
            case "Declined":
                System.out.println("Declined: " + documentId);
                break;
        }
        
        return ResponseEntity.ok("OK");
    }
    
    private String computeHmac(String data, String secret) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKeySpec = new SecretKeySpec(
                secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
            mac.init(secretKeySpec);
            byte[] hash = mac.doFinal(data.getBytes(StandardCharsets.UTF_8));
            return HexFormat.of().formatHex(hash);
        } catch (Exception e) {
            throw new RuntimeException("HMAC computation failed", e);
        }
    }
    
    private void handleCompleted(String documentId) {
        // Download signed PDF
        byte[] pdf = documentApi.downloadDocument(documentId);
        // Save to storage
        
        // Download audit trail
        byte[] audit = documentApi.downloadAuditLog(documentId);
        // Save to storage
    }
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

## Document Status / Properties Migration

### Docusign: GetEnvelope

```java
// Docusign
Envelope envelope = envelopesApi.getEnvelope(accountId, envelopeId);
System.out.println("Status: " + envelope.getStatus());
// Statuses: created, sent, delivered, completed, declined, voided
```

### BoldSign: GetDocumentProperties

```java
// BoldSign
DocumentProperties details = documentApi.getProperties(documentId);
System.out.println("Status: " + details.getStatus());
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

```java
byte[] documents = envelopesApi.getDocument(accountId, envelopeId, "combined");
Files.write(Paths.get("signed-combined.pdf"), documents);
```

### BoldSign

```java
// Download signed document
byte[] pdf = documentApi.downloadDocument(documentId);
Files.write(Paths.get("signed.pdf"), pdf);

// Download audit trail
byte[] audit = documentApi.downloadAuditLog(documentId);
Files.write(Paths.get("audit-trail.pdf"), audit);
```

---

## Remind / Void / Delete Migration

### Docusign

```java
// Remind
envelopesApi.updateRecipients(accountId, envelopeId, /* resend reminder */);

// Void envelope
Envelope voidEnvelope = new Envelope();
voidEnvelope.setStatus("voided");
voidEnvelope.setVoidedReason("No longer needed");
envelopesApi.update(accountId, envelopeId, voidEnvelope);

// Delete (purge)
Envelope purgeEnvelope = new Envelope();
purgeEnvelope.setPurgeState("documents_and_metadata_queued");
envelopesApi.update(accountId, envelopeId, purgeEnvelope);
```

### BoldSign

```java
// Remind specific signer
ReminderMessage reminderMessage = new ReminderMessage();
reminderMessage.setMessage("Please sign this soon");
documentApi.remindDocument("YOUR_DOCUMENT_ID", Arrays.asList("signer@example.com"), reminderMessage);

// Revoke (void) document
RevokeDocument revokeDocument = new RevokeDocument();
revokeDocument.setMessage("This is document revoke message");
documentApi.revokeDocument("YOUR_DOCUMENT_ID", revokeDocument);

// Delete document
documentApi.deleteDocument(documentId);
```

---

## Sender Identity / On-Behalf-Of Migration

### Docusign: Send on behalf of another user

```java
// Docusign: Requires OAuth with specific user's token
// Each user needs their own OAuth consent or JWT sub claim
// The authenticated user IS the sender
```

### BoldSign: Sender Identity (much simpler)

```java
import com.boldsign.api.SenderIdentitiesApi;
import com.boldsign.model.*;

SenderIdentitiesApi senderClient = new SenderIdentitiesApi(apiClient);

// Step 1: Register tenant identity (one-time)
CreateSenderIdentityRequest createRequest = new CreateSenderIdentityRequest();
createRequest.setName("Acme Corp");
createRequest.setEmail("admin@acme.com");
senderClient.createSenderIdentity(createRequest);

// Step 2: Send on behalf of tenant
SendForSign sendRequest = new SendForSign();
sendRequest.setOnBehalfOf("admin@acme.com");  // ← key field
sendRequest.setTitle("Contract from Acme");
sendRequest.setSigners(Arrays.asList(signer));
sendRequest.setFiles(Arrays.asList(new File("contract.pdf")));

documentApi.sendDocument(sendRequest);
```

---

## Signer Authentication Migration

### Docusign

```java
// Docusign: Access code on recipient
Signer signer = new Signer();
signer.setAccessCode("12345");  // Signer must enter this code
// Or phone authentication, ID verification, etc.
```

### BoldSign

```java
// BoldSign: Authentication options on signer
DocumentSigner signer = new DocumentSigner();
signer.setName("John Doe");
signer.setEmailAddress("john@example.com");
signer.setSignerType(DocumentSigner.SignerTypeEnum.SIGNER);
signer.setAuthenticationType(AuthenticationTypeEnum.ACCESS_CODE);
signer.setFormFields(Arrays.asList(signatureField));
// Also supports: EMAIL_OTP, SMS_OTP, ID_VERIFICATION
```

| Docusign Auth | BoldSign Auth | Notes |
|---|---|---|
| Access Code | `AuthenticationTypeEnum.ACCESS_CODE` | Direct equivalent |
| Phone Auth | `AuthenticationTypeEnum.SMS_OTP` | SMS OTP |
| *(No direct equivalent)* | `AuthenticationTypeEnum.EMAIL_OTP` | Email-based OTP |
| ID Verification | `AuthenticationTypeEnum.ID_VERIFICATION` | Passport/Gov ID |
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

### Docusign application.properties

```properties
docusign.integration.key=your-integration-key
docusign.secret.key=your-secret-key
docusign.user.id=your-user-id-guid
docusign.account.id=your-account-id
docusign.auth.server=account-d.docusign.com
docusign.base.path=https://demo.docusign.net/restapi
```

### BoldSign application.properties

```properties
boldsign.api.key=your_api_key_here
boldsign.webhook.secret=your_webhook_secret_here
boldsign.base.url=https://api.boldsign.com
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

```java
public <T> T callWithRetry(Supplier<T> fn, int maxRetries) {
    for (int attempt = 0; attempt <= maxRetries; attempt++) {
        try {
            return fn.get();
        } catch (ApiException e) {
            if (e.getCode() == 429) {
                if (attempt == maxRetries) throw e;
                try {
                    Thread.sleep(60000); // Wait 60 seconds
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException(ie);
                }
            } else {
                throw e;
            }
        }
    }
    throw new RuntimeException("Max retries exceeded");
}
```

---

## Migration Checklist

### Phase 1: Setup
- [ ] Create BoldSign Sandbox account (https://account.boldsign.com/signup?planId=1076)
- [ ] Replace Maven dependency: `com.boldsign:boldsign-java` (not `boldsign-java-sdk`)
- [ ] Replace authentication: Detect DocuSign auth method. If accessToken or Bearer is used then use BoldSign OAuth. Do not switch to API Key unless it is explicitly mentioned.
- [ ] Update base URL configuration
- [ ] Remove all `accountId` references from API calls

### Phase 2: Core Document Workflow
- [ ] Migrate send document flow (EnvelopeDefinition → SendForSign)
- [ ] Convert tabs to form fields (see Tab → FormField mapping)
- [ ] Convert recipients to signers (Signer/Reviewer → DocumentSigner with SignerTypeEnum)
- [ ] Migrate file handling (Base64 → File object)
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
- ❌ Using Base64 encoding — BoldSign requires File objects
- ❌ Not setting `setEnableSigningOrder(true)` when sequential signing is needed
- ❌ Trying to use Docusign `roleName` for templates — BoldSign uses integer `roleIndex`
- ❌ Not verifying webhook `X-BoldSign-Signature` HMAC — required for production security
- ❌ Polling for document status — use webhooks (especially important with 50 req/hour sandbox limit)
- ❌ Passing `accountId` to BoldSign SDK calls — it's not needed and will cause errors
- ❌ Assuming template creation is synchronous — listen for `TemplateCreated` webhook
- ❌ Not handling `Expired` events — Docusign has no equivalent, but BoldSign documents can expire

---

## Reference Links

**BoldSign:**
- Java SDK: https://developers.boldsign.com/sdks/java-sdk/
- Maven Central: https://central.sonatype.com/artifact/com.boldsign/boldsign-java
- GitHub: https://github.com/boldsign/boldsign-java-sdk
- API Reference: https://developers.boldsign.com/
- Webhooks: https://developers.boldsign.com/webhooks/
- Sandbox Signup: https://account.boldsign.com/signup?planId=1076

**Docusign (for reference during migration):**
- REST API Reference: https://developers.docusign.com/docs/esign-rest-api/reference/
- Java SDK: https://github.com/docusign/docusign-esign-java-client
- Tabs Reference: https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/tabs/
- Connect Webhooks: https://developers.docusign.com/platform/webhooks/connect/
