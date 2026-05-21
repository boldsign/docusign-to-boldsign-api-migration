---
name: docusign-to-boldsign-migration-php
description: >
  Complete migration guide for moving from Docusign eSignature API to BoldSign eSignature API
  using PHP. Use this skill whenever a developer mentions migrating from Docusign to BoldSign,
  replacing Docusign with BoldSign, switching eSignature providers, or needs help converting
  Docusign PHP SDK code to BoldSign PHP SDK code. Covers: authentication migration, envelope-to-document
  mapping, recipient-to-signer mapping, tab-to-form-field mapping, embedded signing, templates,
  webhooks (Connect to BoldSign webhooks), sender identity, and complete side-by-side code examples.
  Always trigger this skill for Docusign→BoldSign migration tasks — even if the user just asks
  "Migrate Docusign to BoldSign"
  "how do I convert my Docusign code to BoldSign?" or "what's the BoldSign equivalent of EnvelopesApi?"
-

# Docusign → BoldSign Migration Guide (PHP)

You are now a Docusign-to-BoldSign migration expert for PHP. Your job is to help developers
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
User pastes Docusign PHP code and wants it rewritten for BoldSign
User references docusign/esign-client Composer package and wants BoldSign replacement
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
User needs help with BoldSign .NET/C#, Java, Node.js, or Python migration (those are language-specific skills)

---

## Step 1 — Assess the Existing Docusign Integration

Before writing any migration code, understand what the current Docusign integration does:

1. **Authentication** — JWT Grant, Authorization Code Grant, or legacy header auth?
2. **Core workflows** — Send envelopes, embedded signing, templates, bulk send?
3. **Recipients** — Signers, CC/viewers, in-person signers, routing order?
4. **Tabs** — SignHere, DateSigned, Text, CheckBox, RadioGroup, Dropdown?
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
| Recipient (Signer) | DocumentSigner (SignerType: 'Signer') | Person who signs |
| Recipient (CC) | DocumentCC | Receives copy, no action |
| Recipient (InPersonSigner) | DocumentSigner (SignerType: 'InPersonSigner') | Signs in person |
| Tab | FormField | Fields placed on documents |
| SignHere tab | FormField (FieldType: 'Signature') | Signature placement |
| InitialHere tab | FormField (FieldType: 'Initial') | Initials placement |
| DateSigned tab | FormField (FieldType: 'DateSigned') | Auto-filled date |
| Text tab | FormField (FieldType: 'TextBox') | Free text input |
| CheckBox tab | FormField (FieldType: 'CheckBox') | CheckBox field |
| RadioGroup tab | FormField (FieldType: 'RadioButton') | Radio button selection |
| List/Dropdown tab | FormField (FieldType: 'Dropdown') | Dropdown selection |
| Note tab | FormField (FieldType: 'Label') | Read-only label |
| Attachment tab | FormField (FieldType: 'Attachment') | File attachment |
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

```php
<?php
require_once 'vendor/autoload.php';

use DocuSign\eSign\Client\ApiClient;
use DocuSign\eSign\Api\EnvelopesApi;

// Docusign — OAuth required, accountId needed for every call
$apiClient = new ApiClient();
$apiClient->getConfig()->setHost('https://demo.docusign.net/restapi');
$apiClient->getConfig()->addDefaultHeader('Authorization', "Bearer {$accessToken}");

$envelopesApi = new EnvelopesApi($apiClient);

// accountId is required in EVERY API call
$result = $envelopesApi->createEnvelope($accountId, $envelopeDefinition);
```

### BoldSign: API Key (simplest) or OAuth 2.0

```php
<?php
require_once 'vendor/autoload.php';

use BoldSign\Api\DocumentApi;
use BoldSign\Configuration;

// BoldSign — API Key auth (simplest for server-to-server)
$config = Configuration();
$config->setApiKey('Your-API-Key-Here');
$documentApi = new DocumentApi($config);

// No accountId needed — inferred from API key
$result = $documentApi->sendDocument($sendForSign);
```

```php
// BoldSign — OAuth 2.0 (for user-delegated access)
$config = Configuration();
$config->setAccessToken("Your-Bearer-Token-Here");

$documentApi = new DocumentApi($config);
```

### Key Authentication Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Simplest auth | OAuth JWT (complex) | API Key (single config) |
| Auth configuration | `Authorization: Bearer {token}` | `X-API-KEY: {key}` or `Bearer {token}` |
| Account ID | Required in every API call | Not required (inferred) |
| Base URL (dev) | `https://demo.docusign.net/restapi` | `https://api.boldsign.com` (sandbox toggle) |
| SDK package | `docusign/esign-client` | `boldsign/boldsign-php` |

---

## SDK Installation Migration

### Docusign

```bash
composer require docusign/esign-client
```

### BoldSign

```bash
composer require boldsign/boldsign-php
```

**Packagist:** https://packagist.org/packages/boldsign/boldsign-php  
**SDK Docs:** https://developers.boldsign.com/sdks/php-sdk/

> **Package Name:** The BoldSign SDK for PHP uses `boldsign/boldsign-php` as the Composer package name.

---

## Send Document Migration

### Docusign: Create and Send Envelope

```php
<?php
use DocuSign\eSign\Api\EnvelopesApi;
use DocuSign\eSign\Model\EnvelopeDefinition;
use DocuSign\eSign\Model\Document;
use DocuSign\eSign\Model\Signer;
use DocuSign\eSign\Model\SignHere;
use DocuSign\eSign\Model\Tabs;
use DocuSign\eSign\Model\Recipients;
use DocuSign\eSign\Model\CarbonCopy;

// Setup
$apiClient = new ApiClient();
$apiClient->getConfig()->setHost('https://demo.docusign.net/restapi');
$apiClient->getConfig()->addDefaultHeader('Authorization', "Bearer {$accessToken}");
$envelopesApi = new EnvelopesApi($apiClient);

// Create document
$doc = new Document([
    'document_base64' => base64_encode(file_get_contents('contract.pdf')),
    'name' => 'Contract.pdf',
    'file_extension' => 'pdf',
    'document_id' => '1'
]);

// Create signer with tabs
$signHere = new SignHere([
    'document_id' => '1',
    'page_number' => '1',
    'x_position' => '100',
    'y_position' => '200'
]);

$dateTab = new DateSigned([
    'document_id' => '1',
    'page_number' => '1',
    'x_position' => '300',
    'y_position' => '200'
]);

$textTab = new Text([
    'document_id' => '1',
    'page_number' => '1',
    'x_position' => '100',
    'y_position' => '300',
    'tab_label' => 'company_name',
    'required' => 'true'
]);

$tabs = new Tabs([
    'sign_here_tabs' => [$signHere],
    'date_signed_tabs' => [$dateTab],
    'text_tabs' => [$textTab]
]);

$signer = new Signer([
    'email' => 'john@example.com',
    'name' => 'John Doe',
    'recipient_id' => '1',
    'routing_order' => '1',
    'tabs' => $tabs
]);

// Add CC recipient
$cc = new CarbonCopy([
    'email' => 'manager@example.com',
    'name' => 'Manager',
    'recipient_id' => '2',
    'routing_order' => '2'
]);

$recipients = new Recipients([
    'signers' => [$signer],
    'carbon_copies' => [$cc]
]);

// Build and send envelope
$envelopeDefinition = new EnvelopeDefinition([
    'email_subject' => 'Please sign this document',
    'email_blurb' => 'Please review and sign.',
    'documents' => [$doc],
    'recipients' => $recipients,
    'status' => 'sent'  // "sent" sends immediately
]);

$result = $envelopesApi->createEnvelope($accountId, $envelopeDefinition);
echo "Envelope ID: {$result->getEnvelopeId()}\n";
```

### BoldSign: Send Document (Equivalent)

```php
<?php require_once "vendor/autoload.php";

use BoldSign\Configuration;
use BoldSign\Api\DocumentApi;
use BoldSign\Model\{FormField, Rectangle, DocumentSigner, DocumentCC, SendForSign, FileInfo};

$documentApi = new DocumentApi($config);

// Create form fields (equivalent to Docusign tabs)
$bounds = new Rectangle([
    'x' => 100,
    'y' => 200,
    'width' => 200,
    'height' => 50
]);

$signatureField = new FormField([
    'field_type' => 'Signature',
    'page_number' => 1,
    'bounds' => $bounds,
    'is_required' => true
]);

$dateBounds = new Rectangle([
    'x' => 300,
    'y' => 200,
    'width' => 150,
    'height' => 30
]);

$dateSignedField = new FormField([
    'field_type' => 'DateSigned',
    'page_number' => 1,
    'bounds' => $dateBounds
]);

$textBounds = new Rectangle([
    'x' => 100,
    'y' => 300,
    'width' => 200,
    'height' => 30
]);

$textField = new FormField([
    'id' => 'company_name',
    'field_type' => 'TextBox',
    'page_number' => 1,
    'bounds' => $textBounds,
    'is_required' => true
]);

// Create signer (with form fields attached)
$signer = new DocumentSigner([
    'name' => 'John Doe',
    'email_address' => 'john@example.com',
    'signer_type' => 'Signer',
    'form_fields' => [$signatureField, $dateSignedField, $textField]
]);

// Build and send
$sendRequest = new SendForSign([
    'title' => 'Please sign this document',
    'message' => 'Please review and sign.',
    'signers' => [$signer],
    'cc_details' => [new DocumentCC(['email_address' => 'manager@example.com'])],
    'files' => ['contract.pdf']
]);

$result = $documentApi->sendDocument($sendRequest);
echo "Document ID: {$result->getDocumentId()}\n";
// NOTE: Document send is ASYNC — listen for Sent/SendFailed webhooks
```

### Key Differences When Sending

| Aspect | Docusign | BoldSign |
|---|---|---|
| File format | Base64 string in JSON | SplFileObject (file stream) |
| Tabs/Fields | Attached to recipient's `tabs` property | Attached to signer's `form_fields` array |
| Tab positioning | `x_position`/`y_position` (strings) | `bounds` object (x, y, width, height) |
| CC recipients | Separate `carbon_copies` array | Separate `DocumentCC` recipient |
| Send status | `status: 'sent'` to send immediately | Sends immediately on API call |
| Async behavior | Synchronous (envelope created on return) | **Asynchronous** — must listen for webhooks |
| Subject/Message | `email_subject` / `email_blurb` | `title` / `message` |

---

## Tab → Form Field Mapping (Detailed)

### Docusign Tab Types → BoldSign Field Type

| Docusign Tab | BoldSign Field Type | Notes |
|---|---|---|
| `sign_here_tabs` | `'Signature'` | Direct equivalent |
| `initial_here_tabs` | `'Initial'` | Direct equivalent |
| `date_signed_tabs` | `'DateSigned'` | Auto-populated |
| `text_tabs` | `'TextBox'` | Free-text input |
| `checkbox_tabs` | `'CheckBox'` | Direct equivalent |
| `radio_group_tabs` | `'RadioButton'` | Direct equivalent |
| `list_tabs` | `'Dropdown'` | Dropdown selection |
| `note_tabs` | `'Label'` | Read-only display |
| `attachment_tabs` | `'Attachment'` | File attachment |

### Positioning Conversion

```php
// Docusign: separate X/Y position strings
$signHere = [
    'document_id' => '1',
    'page_number' => '1',
    'x_position' => '100',
    'y_position' => '200'
];

// BoldSign: Rectangle bounds object with x, y, width, height
$bounds = new Rectangle([
    'x' => 100,      // integer, not string
    'y' => 200,
    'width' => 200,
    'height' => 50
]);

$signatureField = new FormField([
    'field_type' => 'Signature',
    'page_number' => 1,  // integer, not string
    'bounds' => $bounds,
    'is_required' => true
]);
```

### Anchor Tags / AutoPlace → Text Tags

```php
// Docusign: Anchor string for auto-placement
$signHere = [
    'anchor_string' => '/sig1/',
    'anchor_x_offset' => '20',
    'anchor_y_offset' => '10',
    'anchor_units' => 'pixels'
];

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

```php
// Docusign: Multiple recipient types
$recipients = new Recipients([
    'signers' => [
        new Signer([
            'email' => 'signer@example.com',
            'name' => 'Signer',
            'recipient_id' => '1',
            'routing_order' => '1'
        ])
    ],
    'carbon_copies' => [
        new CarbonCopy([
            'email' => 'cc@example.com',
            'name' => 'CC Person',
            'recipient_id' => '2',
            'routing_order' => '2'
        ])
    ],
    'in_person_signers' => [
        new InPersonSigner([
            'signer_name' => 'In-Person Signer',
            'email' => 'inperson@example.com',
            'recipient_id' => '3',
            'routing_order' => '3'
        ])
    ]
]);
```

```php
// BoldSign: Unified signer list with signer_type
$signers = [
    new DocumentSigner([
        'name' => 'Signer',
        'email_address' => 'signer@example.com',
        'signer_type' => 'Signer',
        'signer_order' => 1,
        'form_fields' => [$signatureField]
    ]),
    new DocumentSigner([
        'name' => 'Reviewer Person',
        'email_address' => 'cc@example.com',
        'signer_type' => 'Reviewer',
        'signer_order' => 2
    ]),
    new DocumentSigner([
        'name' => 'In-Person Signer',
        'email_address' => 'inperson@example.com',
        'signer_type' => 'InPersonSigner',
        'signer_order' => 3,
        'form_fields' => [$signatureField2]
    ])
];
```

| Docusign Recipient | BoldSign SignerType | Notes |
|---|---|---|
| `Signer` | `'Signer'` | Direct equivalent |
| `CarbonCopy` | `Separate `DocumentCC` recipient | Receives copy, no action |
| `InPersonSigner` | `'InPersonSigner'` | Direct equivalent |
| `CertifiedDelivery` | `'Reviewer'` | Use Viewer (closest equivalent) |
| `Editor` | *(Not supported)* | Not available in BoldSign |
| `Agent` | *(Not supported)* | Not available in BoldSign |
| `Intermediary` | *(Not supported)* | Not available in BoldSign |
| `Witness` | *(Not supported)* | Not available in BoldSign |

---

## Sequential / Parallel Signing Migration

### Docusign: Routing Order

```php
// Docusign: routing_order determines sequence
$signer1 = new Signer(['routing_order' => '1']);
$signer2 = new Signer(['routing_order' => '2']);
// Always enforces routing order when set
```

### BoldSign: Signer Order + EnableSigningOrder

```php
// BoldSign: Must explicitly enable signing order
$signer1 = new DocumentSigner([
    'name' => 'First Signer',
    'email_address' => 'first@example.com',
    'signer_type' => 'Signer',
    'signer_order' => 1,
    'form_fields' => [$field1]
]);

$signer2 = new DocumentSigner([
    'name' => 'Second Signer',
    'email_address' => 'second@example.com',
    'signer_type' => 'Signer',
    'signer_order' => 2,
    'form_fields' => [$field2]
]);

$sendRequest = new SendForSign([
    'title' => 'Sequential Contract',
    'signers' => [$signer1, $signer2],
    'files' => ['contract.pdf'],
    'enable_signing_order' => true  // REQUIRED for sequential signing
]);
```

---

## Embedded Signing Migration

### Docusign: CreateRecipientView

```php
<?php
use DocuSign\eSign\Model\RecipientViewRequest;

// Step 1: Create envelope with client_user_id
$signer = new Signer([
    'email' => 'signer@example.com',
    'name' => 'John Doe',
    'recipient_id' => '1',
    'routing_order' => '1',
    'client_user_id' => '1234'  // Marks as embedded signer
]);

$envelopeDefinition = new EnvelopeDefinition([
    /* ... documents, recipients ... */
    'status' => 'sent'
]);

$result = $envelopesApi->createEnvelope($accountId, $envelopeDefinition);
$envelopeId = $result->getEnvelopeId();

// Step 2: Generate signing URL
$recipientViewRequest = new RecipientViewRequest([
    'authentication_method' => 'none',
    'client_user_id' => '1234',
    'user_name' => 'John Doe',
    'email' => 'signer@example.com',
    'return_url' => 'https://yourapp.com/signing-complete'
]);

$viewUrl = $envelopesApi->createRecipientView($accountId, $envelopeId, $recipientViewRequest);
$signingUrl = $viewUrl->getUrl();  // Expires in 5 minutes
```

### BoldSign: GetEmbeddedSignLink

```php
<?php
// Step 1: Send document normally
$signer = new DocumentSigner([
    'name' => 'John Doe',
    'email_address' => 'john@example.com',
    'signer_type' => 'Signer',
    'form_fields' => [$signatureField]
]);

$sendRequest = new SendForSign([
    'title' => 'Contract for Signing',
    'signers' => [$signer],
    'files' => ['contract.pdf']
]);

$result = $documentApi->sendDocument($sendRequest);
$documentId = $result->getDocumentId();

// Step 2: Generate embedded signing URL
$signLinkResult = $documentApi->getEmbeddedSignLink(
    $documentId,
    'john@example.com',
    null,   // countryCode (optional, for SMS OTP)
    null,   // sendSMS
    null, // signLinkValidTill
    'https://yourapp.com/signing-complete' // redirectUrl
);

$signingUrl = $signLinkResult->getSignLink();
// Embed in <iframe> or redirect user to this URL
```

### Key Embedded Signing Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Embedded marker | `client_user_id` required on signer | No special marker needed |
| URL generation | `createRecipientView` with matching credentials | `getEmbeddedSignLink` with documentId + email |
| URL expiry | 5 minutes (default) | Configurable via `signLinkValidTill` |
| Match requirements | user_name + email + client_user_id must match | signerEmail must match |

---

## Embedded Send (Sender View) Migration

### Docusign: CreateSenderView

```php
// Docusign: Create draft envelope, then get sender view
$envelopeDefinition->setStatus('created');  // Draft
$result = $envelopesApi->createEnvelope($accountId, $envelopeDefinition);

$senderViewRequest = new ReturnUrlRequest(['return_url' => 'https://yourapp.com/send-complete']);
$senderView = $envelopesApi->createSenderView($accountId, $result->getEnvelopeId(), $senderViewRequest);
$senderUrl = $senderView->getUrl();
```

### BoldSign: CreateEmbeddedRequestUrl

```php
// BoldSign: Create embedded send request
$signer = new DocumentSigner([
    'name' => 'Signer',
    'email_address' => 'signer@example.com',
    'signer_type' => 'Signer'
]);

$embeddedRequest = new EmbeddedDocumentRequest([
    'title' => 'Contract',
    'signers' => [$signer],
    'files' => ['contract.pdf'],
    'show_toolbar' => true,
    'show_save_button' => true,
    'show_send_button' => true,
    'show_navigation_buttons' => true,
    'redirect_url' => 'https://yourapp.com/send-complete'
]);

$result = $documentApi->createEmbeddedRequestUrlDocument($embeddedRequest);
$senderUrl = $result->getSendUrl();
// Embed in <iframe>
```

---

## Template Migration

### Docusign: Create Envelope from Template

```php
use DocuSign\eSign\Model\TemplateRole;

$templateRole = new TemplateRole([
    'email' => 'signer@example.com',
    'name' => 'John Doe',
    'role_name' => 'Signer'  // Must match template role name
]);

$envelopeDefinition = new EnvelopeDefinition([
    'template_id' => 'DOCUSIGN_TEMPLATE_ID',
    'template_roles' => [$templateRole],
    'status' => 'sent'
]);

$result = $envelopesApi->createEnvelope($accountId, $envelopeDefinition);
```

### BoldSign: Send from Template

```php
use BoldSign\Api\TemplateApi;
use BoldSign\Model\SendForSignFromTemplateForm;
use BoldSign\Model\Roles;

$templateApi = new TemplateApi(null, $config);

$role = new Roles([
    'role_index' => 1,   // Index-based (1, 2, 3...), not name-based
    'signer_name' => 'John Doe',
    'signer_email' => 'signer@example.com'
]);

$send_for_sign_from_template = new SendForSignFromTemplateForm(['roles' => [$role]]);
$result = $templateApi->sendUsingTemplate($template_id = 'YOUR_TEMPLATE_ID', $send_for_sign_from_template);
echo "Document ID: {$result->getDocumentId()}\n";
```

### Key Template Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Role mapping | By `role_name` (string match) | By `role_index` (integer, 1-based) |
| Template creation | Synchronous | **Asynchronous** — listen for webhook |
| SDK class | `EnvelopesApi` with template params | Dedicated `TemplateApi` |

---

## Webhook Migration

### Docusign: EventNotification

```php
$eventNotification = [
    'url' => 'https://yourapp.com/webhooks/docusign',
    'require_acknowledgment' => 'true',
    'logging_enabled' => 'true',
    'envelope_events' => [
        ['envelope_event_status_code' => 'completed'],
        ['envelope_event_status_code' => 'declined'],
        ['envelope_event_status_code' => 'voided']
    ],
    'recipient_events' => [
        ['recipient_event_status_code' => 'Completed'],
        ['recipient_event_status_code' => 'Declined']
    ]
];

$envelopeDefinition['event_notification'] = $eventNotification;
```

### BoldSign: Webhook Handler

```php
<?php
// webhook.php

function verifyHmac(string $payload, string $signature): bool
{
    $secret = getenv('BOLDSIGN_WEBHOOK_SECRET');
    $expected = hash_hmac('sha256', $payload, $secret);
    return hash_equals($expected, $signature);
}

$rawBody = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_BOLDSIGN_SIGNATURE'] ?? '';

if (!verifyHmac($rawBody, $signature)) {
    http_response_code(401);
    exit('Invalid signature');
}

$payload = json_decode($rawBody, true);
$eventType = $payload['event']['eventType'];
$documentId = $payload['document']['documentId'];

switch ($eventType) {
    case 'Completed':
        handleCompleted($documentId);
        break;
    case 'SendFailed':
        error_log("Send failed for $documentId: " . json_encode($payload['document']));
        break;
    case 'Declined':
        error_log("Declined: $documentId");
        break;
}

http_response_code(200);
echo 'OK';

function handleCompleted(string $documentId): void
{
    global $documentApi;

    // Download signed PDF
    $pdf = $documentApi->downloadDocument($documentId);
    file_put_contents("signed/{$documentId}.pdf", $pdf);

    // Download audit trail
    $audit = $documentApi->downloadAuditLog($documentId);
    file_put_contents("signed/{$documentId}-audit.pdf", $audit);
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

```php
$envelope = $envelopesApi->getEnvelope($accountId, $envelopeId);
echo "Status: {$envelope->getStatus()}\n";
// Statuses: created, sent, delivered, completed, declined, voided
```

### BoldSign: GetDocumentProperties

```php
$details = $documentApi->getProperties($documentId);
echo "Status: {$details->getStatus()}\n";
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

```php
$documents = $envelopesApi->getDocument($accountId, $envelopeId, 'combined');
file_put_contents('signed-combined.pdf', $documents);
```

### BoldSign

```php
// Download signed document
$pdf = $documentApi->downloadDocument($documentId);
file_put_contents('signed.pdf', $pdf);

// Download audit trail
$audit = $documentApi->downloadAuditLog($documentId);
file_put_contents('audit-trail.pdf', $audit);
```

---

## Remind / Void / Delete Migration

### Docusign

```php
// Remind
$envelopesApi->updateRecipients($accountId, $envelopeId, /* resend reminder */);

// Void envelope
$envelope = new Envelope();
$envelope->setStatus('voided');
$envelope->setVoidedReason('No longer needed');
$envelopesApi->update($accountId, $envelopeId, $envelope);

// Delete (purge)
$envelope = new Envelope();
$envelope->setPurgeState('documents_and_metadata_queued');
$envelopesApi->update($accountId, $envelopeId, $envelope);
```

### BoldSign

```php
// Remind specific signer
$reminder_message = new ReminderMessage([
    'message' => 'Please sign this soon'
]);

$documentApi->remindDocument($document_id = 'YOUR_DOCUMENT_ID', $receiver_emails = ['signer@example.com'], $reminder_message);

// Revoke (void) document
$revoke_document = new RevokeDocument([
    'message' => 'This is document revoke message'
]);
$documentApi->revokeDocument($document_id = 'YOUR_DOCUMENT_ID', $revoke_document);

// Delete document
$documentApi->deleteDocument($documentId);
```

---

## Sender Identity / On-Behalf-Of Migration

### Docusign: Send on behalf of another user

```php
// Docusign: Requires OAuth with specific user's token
// Each user needs their own OAuth consent or JWT sub claim
// The authenticated user IS the sender
```

### BoldSign: Sender Identity (much simpler)

```php
// BoldSign: Register identity once, then use onBehalfOf
$senderClient = new SenderIdentityClient(null, $config);

// Step 1: Register tenant identity (one-time)
$senderClient->createSenderIdentity(new CreateSenderIdentityRequest([
    'name' => 'Acme Corp',
    'email' => 'admin@acme.com'
]));

// Step 2: Send on behalf of tenant
$sendRequest = new SendForSign([
    'on_behalf_of' => 'admin@acme.com',  // ← key field
    'title' => 'Contract from Acme',
    'signers' => [$signer],
    'files' => ['contract.pdf']
]);

$documentApi->sendDocument($sendRequest);
```

---

## Signer Authentication Migration

### Docusign

```php
// Docusign: Access code on recipient
$signer = new Signer([
    'access_code' => '12345',  // Signer must enter this code
]);
// Or phone authentication, ID verification, etc.
```

### BoldSign

```php
// BoldSign: Authentication options on signer
$signer = new DocumentSigner([
    'name' => 'John Doe',
    'email_address' => 'john@example.com',
    'signer_type' => 'Signer',
    'authentication_type' => 'AccessCode',
    'form_fields' => [$signatureField]
]);
// Also supports: EmailOTP, SMSOTP, IdVerification
```

| Docusign Auth | BoldSign Auth | Notes |
|---|---|---|
| Access Code | `'AccessCode'` | Direct equivalent |
| Phone Auth | `'SMSOTP'` | SMS OTP |
| *(No direct equivalent)* | `'EmailOTP'` | Email-based OTP |
| ID Verification | `'IdVerification'` | Passport/Gov ID |
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

```php
function callWithRetry(callable $fn, int $maxRetries = 1)
{
    for ($attempt = 0; $attempt <= $maxRetries; $attempt++) {
        try {
            return $fn();
        } catch (ApiException $e) {
            if ($e->getCode() == 429) {
                if ($attempt == $maxRetries) throw $e;
                sleep(60); // Wait 60 seconds
            } else {
                throw $e;
            }
        }
    }
    throw new Exception('Max retries exceeded');
}
```

---

## Laravel Integration Example

```php
// routes/api.php
Route::post('/webhooks/boldsign', [BoldSignWebhookController::class, 'handle']);

// app/Http/Controllers/BoldSignWebhookController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Log;

class BoldSignWebhookController extends Controller
{
    public function handle(Request $request): Response
    {
        $signature = $request->header('X-BoldSign-Signature');
        $secret = config('services.boldsign.webhook_secret');
        $expected = hash_hmac('sha256', $request->getContent(), $secret);

        if (!hash_equals($expected, $signature)) {
            abort(401);
        }

        $payload = $request->json()->all();
        $eventType = $payload['event']['eventType'];

        match($eventType) {
            'Completed' => $this->handleCompleted($payload),
            'SendFailed' => Log::error('BoldSign send failed', $payload),
            'Declined' => Log::info('Document declined', $payload),
            default => null,
        };

        return response('OK', 200);
    }

    private function handleCompleted(array $payload): void
    {
        // Download and store signed document
        $documentId = $payload['document']['documentId'];
        // ... handle completion logic
    }
}
```

---

## Migration Checklist

### Phase 1: Setup
- [ ] Create BoldSign Sandbox account (https://account.boldsign.com/signup?planId=1076)
- [ ] Replace Composer package: `composer require boldsign/boldsign-php`
- [ ] Replace authentication: Detect DocuSign auth method. If accessToken or Bearer is used then use BoldSign OAuth. Do not switch to API Key unless it is explicitly mentioned.
- [ ] Update base URL configuration
- [ ] Remove all `accountId` parameters from API calls

### Phase 2: Core Document Workflow
- [ ] Migrate send document flow (EnvelopeDefinition → SendForSign)
- [ ] Convert tabs to form fields (see Tab → FormField mapping)
- [ ] Convert recipients to signers (Signer/Reviewer → DocumentSigner with SignerType)
- [ ] Migrate file handling (Base64 → SplFileObject)
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
- ❌ Using Base64 encoding — BoldSign requires file objects
- ❌ Not setting `enable_signing_order = true` for sequential signing
- ❌ Using Docusign `role_name` — BoldSign uses integer `role_index`
- ❌ Not verifying webhook HMAC signatures
- ❌ Polling for document status instead of using webhooks
- ❌ Passing `accountId` to BoldSign API calls
- ❌ Assuming template creation is synchronous — listen for `TemplateCreated` webhook
- ❌ Not handling `Expired` events — Docusign has no equivalent, but BoldSign documents can expire

---

## Reference Links

**BoldSign:**
- PHP SDK: https://developers.boldsign.com/sdks/php-sdk/
- Composer: https://packagist.org/packages/boldsign/boldsign-php
- GitHub: https://github.com/boldsign/boldsign-php-sdk
- API Reference: https://developers.boldsign.com/
- Webhooks: https://developers.boldsign.com/webhooks/
- Sandbox Signup: https://account.boldsign.com/signup?planId=1076

**Docusign (for reference during migration):**
- REST API Reference: https://developers.docusign.com/docs/esign-rest-api/reference/
- PHP SDK: https://github.com/docusign/docusign-esign-php-client
- Tabs Reference: https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/tabs/
- Connect Webhooks: https://developers.docusign.com/platform/webhooks/connect/
