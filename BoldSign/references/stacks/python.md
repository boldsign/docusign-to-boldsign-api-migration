---
name: docusign-to-boldsign-migration-python
description: >
  Complete migration guide for moving from Docusign eSignature API to BoldSign eSignature API
  using Python. Use this skill whenever a developer mentions migrating from Docusign to BoldSign,
  replacing Docusign with BoldSign, switching eSignature providers, or needs help converting
  Docusign Python SDK code to BoldSign Python SDK code. Covers: authentication migration, envelope-to-document
  mapping, recipient-to-signer mapping, tab-to-form-field mapping, embedded signing, templates,
  webhooks (Connect to BoldSign webhooks), sender identity, and complete side-by-side code examples.
  Always trigger this skill for Docusign→BoldSign migration tasks — even if the user just asks
  "Migrate Docusign to BoldSign"
  "how do I convert my Docusign code to BoldSign?" or "what's the BoldSign equivalent of EnvelopesApi?"
-

# Docusign → BoldSign Migration Guide (Python)

You are now a Docusign-to-BoldSign migration expert for Python. Your job is to help developers
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
User pastes Docusign Python code and wants it rewritten for BoldSign
User references docusign/docusign-esign-python-client package and wants BoldSign replacement
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
User needs help with BoldSign .NET/C#, Java, Node.js, or PHP migration (those are language-specific skills)

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
| Recipient (Signer) | DocumentSigner (SignerType: 'Signer') | Person who signs |
| Recipient (CC) | DocumentCC | Receives copy, no action |
| Recipient (InPersonSigner) | DocumentSigner (SignerType: 'InPersonSigner') | Signs in person |
| Tab | FormField | Fields placed on documents |
| SignHere tab | FormField (FieldType: 'Signature') | Signature placement |
| InitialHere tab | FormField (FieldType: 'Initial') | Initials placement |
| DateSigned tab | FormField (FieldType: 'DateSigned') | Auto-filled date |
| Text tab | FormField (FieldType: 'TextBox') | Free text input |
| Checkbox tab | FormField (FieldType: 'Checkbox') | Checkbox field |
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

```python
from docusign_esign import ApiClient
from docusign_esign.apis import EnvelopesApi

# Docusign — OAuth required, accountId needed for every call
api_client = ApiClient()
api_client.host = 'https://demo.docusign.net/restapi'
api_client.set_default_header('Authorization', f'Bearer {access_token}')

envelopes_api = EnvelopesApi(api_client)

# accountId is required in EVERY API call
result = envelopes_api.create_envelope(account_id, envelope_definition)
```

### BoldSign: API Key (simplest) or OAuth 2.0

```python

# BoldSign — API Key auth (simplest for server-to-server)
import boldsign
configuration = boldsign.Configuration(api_key="YOUR_API_KEY")
with boldsign.ApiClient(configuration) as api_client:

document_api = boldsign.DocumentApi(api_client)

# No accountId needed — inferred from API key
result = document_api.send_document(send_for_sign)
```

```python
# BoldSign — OAuth 2.0 (for user-delegated access)
configuration = boldsign.Configuration(access_token="Your-Bearer-Token-Here")
with boldsign.ApiClient(configuration) as api_client:

document_api = boldsign.DocumentApi(api_client)
```

> **Note:** BoldSign Python SDK uses `boldsign` for pip installation but imports from the same module name for consistency with other SDKs.

### Key Authentication Differences

| Aspect | Docusign | BoldSign |
|---|---|---|
| Simplest auth | OAuth JWT (complex) | API Key (single config) |
| Auth configuration | `Authorization: Bearer {token}` | `X-API-KEY: {key}` or `Bearer {token}` |
| Account ID | Required in every API call | Not required (inferred) |
| Base URL (dev) | `https://demo.docusign.net/restapi` | `https://api.boldsign.com` (sandbox toggle) |
| SDK package | `docusign-esign` | `boldsign` |

---

## SDK Installation Migration

### Docusign

```bash
pip install docusign-esign
```

### BoldSign

```bash
pip install boldsign
```

**PyPI:** https://pypi.org/project/boldsign/  
**SDK Docs:** https://developers.boldsign.com/sdks/python-sdk/

> **Package Name:** The BoldSign SDK for Python uses `boldsign` as the pip package name with consistent imports.

---

## Send Document Migration

### Docusign: Create and Send Envelope

```python
from docusign_esign.apis import EnvelopesApi
from docusign_esign.models import (
    EnvelopeDefinition, Document, Signer, SignHere, Tabs,
    Recipients, CarbonCopy, DateSigned, Text
)

# Setup
api_client = ApiClient()
api_client.host = 'https://demo.docusign.net/restapi'
api_client.set_default_header('Authorization', f'Bearer {access_token}')
envelopes_api = EnvelopesApi(api_client)

# Create document
with open('contract.pdf', 'rb') as doc_file:
    doc_base64 = base64.b64encode(doc_file.read()).decode('utf-8')

doc = Document(
    document_base64=doc_base64,
    name='Contract.pdf',
    file_extension='pdf',
    document_id='1'
)

# Create tabs
sign_here = SignHere(
    document_id='1',
    page_number='1',
    x_position='100',
    y_position='200'
)

date_tab = DateSigned(
    document_id='1',
    page_number='1',
    x_position='300',
    y_position='200'
)

text_tab = Text(
    document_id='1',
    page_number='1',
    x_position='100',
    y_position='300',
    tab_label='company_name',
    required='true'
)

tabs = Tabs(
    sign_here_tabs=[sign_here],
    date_signed_tabs=[date_tab],
    text_tabs=[text_tab]
)

# Create signer with tabs
signer = Signer(
    email='john@example.com',
    name='John Doe',
    recipient_id='1',
    routing_order='1',
    tabs=tabs
)

# Create CC recipient
cc = CarbonCopy(
    email='manager@example.com',
    name='Manager',
    recipient_id='2',
    routing_order='2'
)

recipients = Recipients(
    signers=[signer],
    carbon_copies=[cc]
)

# Build and send envelope
envelope_definition = EnvelopeDefinition(
    email_subject='Please sign this document',
    email_blurb='Please review and sign.',
    documents=[doc],
    recipients=recipients,
    status='sent'  # "sent" sends immediately
)

result = envelopes_api.create_envelope(account_id, envelope_definition)
print(f"Envelope ID: {result.envelope_id}")
```

### BoldSign: Send Document (Equivalent)

```python
import boldsign

with boldsign.ApiClient(configuration) as api_client:
    document_api = boldsign.DocumentApi(api_client)

# Create form fields (equivalent to Docusign tabs)
bounds = boldsign.Rectangle(x=100, y=200, width=200, height=50)

signature_field = boldsign.FormField(
    field_type='Signature',
    page_number=1,
    bounds=bounds,
    is_required=True
)

date_bounds = boldsign.Rectangle(x=300, y=200, width=150, height=30)

date_signed_field = boldsign.FormField(
    field_type='DateSigned',
    page_number=1,
    bounds=date_bounds
)

text_bounds = boldsign.Rectangle(x=100, y=300, width=200, height=30)

text_field = boldsign.FormField(
    id='company_name',
    field_type='TextBox',
    page_number=1,
    bounds=text_bounds,
    is_required=True
)

# Create signer (with form fields attached)
signer = boldsign.DocumentSigner(
    name='John Doe',
    email_address='john@example.com',
    signer_type='Signer',
    form_fields=[signature_field, date_signed_field, text_field]
)

# Add viewer (equivalent to docusign CC)
cc_recipient = boldsign.DocumentCC(
    name='Manager',
    email_address='manager@example.com'
)

# Build and send
send_request = boldsign.SendForSign(
    title='Please sign this document',
    message='Please review and sign.',
    signers=[signer],
    cc=[cc_recipient],
    files=["FILE_PATH"]
)

result = document_api.send_document(send_request)

print(f"Document ID: {result.document_id}")
# NOTE: Document send is ASYNC — listen for Sent/SendFailed webhooks
```

### Key Differences When Sending

| Aspect | Docusign | BoldSign |
|---|---|---|
| File format | Base64 string in JSON | File object (binary stream) |
| Tabs/Fields | Attached to recipient's `tabs` property | Attached to signer's `form_fields` array |
| Tab positioning | `x_position`/`y_position` (strings) | `bounds` object (x, y, width, height integers) |
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
| `checkbox_tabs` | `'Checkbox'` | Direct equivalent |
| `radio_group_tabs` | `'RadioButton'` | Direct equivalent |
| `list_tabs` | `'Dropdown'` | Dropdown selection |
| `note_tabs` | `'Label'` | Read-only display |
| `attachment_tabs` | `'Attachment'` | File attachment |

### Positioning Conversion

```python
# Docusign: separate X/Y position strings
sign_here = {
    'document_id': '1',
    'page_number': '1',
    'x_position': '100',
    'y_position': '200'
}

# BoldSign: Rectangle bounds object with x, y, width, height
import boldsign
bounds = boldsign.Rectangle(
    x=100,      # integer, not string
    y=200,
    width=200,
    height=50
)

signature_field = boldsign.FormField(
    field_type='Signature',
    page_number=1,  # integer, not string
    bounds=bounds,
    is_required=True
)
```

### Anchor Tags / AutoPlace → Text Tags

```python
# Docusign: Anchor string for auto-placement
sign_here = {
    'anchor_string': '/sig1/',
    'anchor_x_offset': '20',
    'anchor_y_offset': '10',
    'anchor_units': 'pixels'
}

# BoldSign: Text Tags (embed in document before upload)
# Syntax: {{*Field type*|*Signer Index*|*Required*|*Field label*|*Field ID*}}
# In your Word/PDF document, add invisible text:
#   {{sign|signer_index}}    → Signature field
#   {{init|signer_index}}    → Initials field
#   {{date|signer_index}}    → Date signed
#   {{text|signer_index}}    → Text input
# BoldSign auto-converts these to real fields on upload.
# Reference: https://developers.boldsign.com/documents/text-tags/
```

---

## Recipient Type Migration

```python
# Docusign: Multiple recipient types
recipients = Recipients(
    signers=[
        Signer(
            email='signer@example.com',
            name='Signer',
            recipient_id='1',
            routing_order='1'
        )
    ],
    carbon_copies=[
        CarbonCopy(
            email='cc@example.com',
            name='CC Person',
            recipient_id='2',
            routing_order='2'
        )
    ],
    in_person_signers=[
        InPersonSigner(
            signer_name='In-Person Signer',
            email='inperson@example.com',
            recipient_id='3',
            routing_order='3'
        )
    ]
)
```

```python
# BoldSign: Unified signer list with signer_type
signers = [
    DocumentSigner(
        name='Signer',
        email_address='signer@example.com',
        signer_type='Signer',
        signer_order=1,
        form_fields=[signature_field]
    ),
    DocumentSigner(
        name='Reviewer Person',
        email_address='cc@example.com',
        signer_type='Reviewer',
        signer_order=2
    ),
    DocumentSigner(
        name='In-Person Signer',
        email_address='inperson@example.com',
        signer_type='InPersonSigner',
        signer_order=3,
        form_fields=[signature_field2]
    )
]
```

| Docusign Recipient | BoldSign SignerType | Notes |
|---|---|---|
| `Signer` | `'Signer'` | Direct equivalent |
| `CarbonCopy` | Separate `DocumentCC` recipient | Receives copy, no action |
| `InPersonSigner` | `'InPersonSigner'` | Direct equivalent |
| `CertifiedDelivery` | `SignerType.Reviewer` | Use Viewer (closest equivalent) |
| `Editor` | *(Not supported)* | Not available in BoldSign |
| `Agent` | *(Not supported)* | Not available in BoldSign |
| `Intermediary` | *(Not supported)* | Not available in BoldSign |
| `Witness` | *(Not supported)* | Not available in BoldSign |

---

## Sequential / Parallel Signing Migration

### Docusign: Routing Order

```python
# Docusign: routing_order determines sequence
signer1 = Signer(routing_order='1')
signer2 = Signer(routing_order='2')
# Always enforces routing order when set
```

### BoldSign: Signer Order + EnableSigningOrder

```python
# BoldSign: Must explicitly enable signing order
signer1 = DocumentSigner(
    name='First Signer',
    email_address='first@example.com',
    signer_type='Signer',
    signer_order=1,
    form_fields=[field1]
)

signer2 = DocumentSigner(
    name='Second Signer',
    email_address='second@example.com',
    signer_type='Signer',
    signer_order=2,
    form_fields=[field2]
)

send_request = SendForSign(
    title='Sequential Contract',
    signers=[signer1, signer2],
    files=["contract.pdf"],
    enable_signing_order=True  # REQUIRED for sequential signing
)
```

---

## Embedded Signing Migration

### Docusign: CreateRecipientView

```python
from docusign_esign.models import RecipientViewRequest

# Step 1: Create envelope with client_user_id
signer = Signer(
    email='signer@example.com',
    name='John Doe',
    recipient_id='1',
    routing_order='1',
    client_user_id='1234'  # Marks as embedded signer
)

envelope_definition = EnvelopeDefinition(
    # ... documents, recipients ...
    status='sent'
)

result = envelopes_api.create_envelope(account_id, envelope_definition)
envelope_id = result.envelope_id

# Step 2: Generate signing URL
recipient_view_request = RecipientViewRequest(
    authentication_method='none',
    client_user_id='1234',
    user_name='John Doe',
    email='signer@example.com',
    return_url='https://yourapp.com/signing-complete'
)

view_url = envelopes_api.create_recipient_view(
    account_id, envelope_id, recipient_view_request
)
signing_url = view_url.url
```

### BoldSign: GetEmbeddedSignLink

```python
# Step 1: Send document normally
signer = DocumentSigner(
    name='John Doe',
    email_address='john@example.com',
    signer_type='Signer',
    form_fields=[signature_field]
)

send_request = SendForSign(
    title='Contract for Signing',
    signers=[signer],
    files=['contract.pdf']
)

result = document_api.send_document(send_request)
document_id = result.document_id

# Step 2: Generate embedded signing URL

sign_link_result = document_api.get_embedded_sign_link(
    document_id=document_id,
    signer_email='john@example.com',
    redirect_url='https://yourapp.com/signing-complete'
)

signing_url = sign_link_result.sign_link
# Embed in <iframe> or redirect user to this URL
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

```python
# Docusign: Create draft envelope, then get sender view
envelope_definition.status = 'created'  # Draft
result = envelopes_api.create_envelope(account_id, envelope_definition)

sender_view_request = ReturnUrlRequest(
    return_url='https://yourapp.com/send-complete'
)
sender_view = envelopes_api.create_sender_view(
    account_id, result.envelope_id, sender_view_request
)
sender_url = sender_view.url
```

### BoldSign: CreateEmbeddedRequestUrl

```python
# BoldSign: Create embedded send request
signer = boldsign.DocumentSigner(
    name='Signer',
    email_address='signer@example.com',
    signer_type='Signer'
)

embedded_request = boldsign.EmbeddedDocumentRequest(
    title='Contract',
    signers=[signer],
    files=["FILE_PATH"],
    show_toolbar=True,
    show_save_button=True,
    show_send_button=True,
    show_navigation_buttons=True,
    redirect_url='https://yourapp.com/send-complete'
)

result = document_api.create_embedded_request_url_document(embedded_document_request=embedded_request)
sender_url = result.send_url
# Embed in <iframe>
```

---

## Template Migration

### Docusign: Create Envelope from Template

```python
from docusign_esign.models import TemplateRole

template_role = TemplateRole(
    email='signer@example.com',
    name='John Doe',
    role_name='Signer'  # Must match template role name
)

envelope_definition = EnvelopeDefinition(
    template_id='DOCUSIGN_TEMPLATE_ID',
    template_roles=[template_role],
    status='sent'
)

result = envelopes_api.create_envelope(account_id, envelope_definition)
```

### BoldSign: Send from Template

```python
import boldsign

with boldsign.ApiClient(configuration) as api_client:

template_api = boldsign.TemplateApi(api_client)
role = boldsign.Role(
	roleIndex=1, # Index-based (1, 2, 3...), not name-based
	signer_name="John Doe",
	signer_email="signer@example.com",
	signerType='Signer')

send_for_sign_from_template = boldsign.SendForSignFromTemplateForm(roles=[role])

result = template_api.send_using_template(template_id = "YOUR_TEMPLATE_ID", send_for_sign_from_template_form=send_for_sign_from_template)
print(f"Document ID: {result.document_id}")
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

```python
event_notification = {
    'url': 'https://yourapp.com/webhooks/docusign',
    'require_acknowledgment': 'true',
    'logging_enabled': 'true',
    'envelope_events': [
        {'envelope_event_status_code': 'completed'},
        {'envelope_event_status_code': 'declined'},
        {'envelope_event_status_code': 'voided'}
    ],
    'recipient_events': [
        {'recipient_event_status_code': 'Completed'},
        {'recipient_event_status_code': 'Declined'}
    ]
}

envelope_definition['event_notification'] = event_notification
```

### BoldSign: Webhook Handler (Flask Example)

```python
# webhook.py
from flask import Flask, request
import hmac
import hashlib
import json
import logging

app = Flask(__name__)
WEBHOOK_SECRET = os.getenv('BOLDSIGN_WEBHOOK_SECRET')

def verify_hmac(payload: bytes, signature: str) -> bool:
    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

@app.route('/webhooks/boldsign', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-BoldSign-Signature', '')
    payload = request.get_data()

    if not verify_hmac(payload, signature):
        return 'Invalid signature', 401

    data = json.loads(payload)
    event_type = data['event']['eventType']
    document_id = data['document']['documentId']

    if event_type == 'Completed':
        handle_completed(document_id)
    elif event_type == 'SendFailed':
        logging.error(f"Send failed for {document_id}: {data['document']}")
    elif event_type == 'Declined':
        logging.info(f"Declined: {document_id}")

    return 'OK', 200

def handle_completed(document_id: str) -> None:
    # Download signed PDF
    pdf = document_api.download_document(document_id)
    with open(f'signed/{document_id}.pdf', 'wb') as f:
        f.write(pdf)

    # Download audit trail
    audit = document_api.download_audit_log(document_id)
    with open(f'signed/{document_id}-audit.pdf', 'wb') as f:
        f.write(audit)

if __name__ == '__main__':
    app.run(debug=True)
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

```python
envelope = envelopes_api.get_envelope(account_id, envelope_id)
print(f"Status: {envelope.status}")
# Statuses: created, sent, delivered, completed, declined, voided
```

### BoldSign: GetDocumentProperties

```python
details = document_api.get_properties(document_id)
print(f"Status: {details.status}")
# Statuses: InProgress, Completed, Declined, Expired, Revoked
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

```python
documents = envelopes_api.get_document(account_id, envelope_id, 'combined')
with open('signed-combined.pdf', 'wb') as f:
    f.write(documents)
```

### BoldSign

```python
# Download signed document
pdf = document_api.download_document(document_id)
with open('signed.pdf', 'wb') as f:
    f.write(pdf)

# Download audit trail
audit = document_api.download_audit_log(document_id)
with open('audit-trail.pdf', 'wb') as f:
    f.write(audit)
```

---

## Remind / Void / Delete Migration

### Docusign

```python
# Remind
envelopes_api.update_recipients(account_id, envelope_id, /* resend reminder */)

# Void envelope
envelope = Envelope()
envelope.status = 'voided'
envelope.voided_reason = 'No longer needed'
envelopes_api.update(account_id, envelope_id, envelope)

# Delete (purge)
envelope = Envelope()
envelope.purge_state = 'documents_and_metadata_queued'
envelopes_api.update(account_id, envelope_id, envelope)
```

### BoldSign

```python
# Remind specific signer
reminder_message = boldsign.ReminderMessage(message="Please sign this soon")
document_api.remind_document(document_id="YOUR_DOCUMENT_ID", reminder_message=reminder_message)

# Revoke (void) document
revoke_document = boldsign.RevokeDocument(message="This is document revoke message")
document_api.revoke_document(document_id="YOUR_DOCUMENT_ID", revoke_document=revoke_document)

# Delete document
document_api.delete_document(document_id)
```

---

## Sender Identity / On-Behalf-Of Migration

### Docusign: Send on behalf of another user

```python
# Docusign: Requires OAuth with specific user's token
# Each user needs their own OAuth consent or JWT sub claim
# The authenticated user IS the sender
```

### BoldSign: Sender Identity (much simpler)

```python
# BoldSign: Register identity once, then use onBehalfOf
# Step 1: Register tenant identity (one-time)
sender_identities_api = boldsign.SenderIdentitiesApi(api_client)
    sender_identity_request = boldsign.CreateSenderIdentityRequest(
        name='Acme Corp',
        mail='admin@acme.com'
    )
    sender_identities_api.create_sender_identities(sender_identity_request)

# Step 2: Send on behalf of tenant
send_request = SendForSign(
    on_behalf_of='admin@acme.com',  # ← key field
    title='Contract from Acme',
    signers=[signer],
    files=["FILE_PATH"]
)

document_api.send_document(send_request)
```

---

## Signer Authentication Migration

### Docusign

```python
# Docusign: Access code on recipient
signer = Signer(
    access_code='12345'  # Signer must enter this code
)
# Or phone authentication, ID verification, etc.
```

### BoldSign

```python
# BoldSign: Authentication options on signer
signer = DocumentSigner(
    name='John Doe',
    email_address='john@example.com',
    signer_type='Signer',
    authentication_type='AccessCode',
    form_fields=[signature_field]
)
# Also supports: EmailOTP, SMSOTP, IdVerification
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

```python
import time

def call_with_retry(func, *args, max_retries=1, **kwargs):
    for attempt in range(max_retries + 1):
        try:
            return func(*args, **kwargs)
        except ApiException as e:
            if e.status == 429:
                if attempt == max_retries:
                    raise
                time.sleep(60)  # Wait 60 seconds
            else:
                raise
    raise Exception('Max retries exceeded')
```

---

## FastAPI Integration Example

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import hmac
import hashlib
import json
import logging

app = FastAPI()
WEBHOOK_SECRET = os.getenv('BOLDSIGN_WEBHOOK_SECRET')

def verify_hmac(payload: bytes, signature: str) -> bool:
    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

@app.post('/webhooks/boldsign')
async def handle_webhook(request: Request):
    signature = request.headers.get('X-BoldSign-Signature', '')
    payload = await request.body()

    if not verify_hmac(payload, signature):
        return JSONResponse({'error': 'Invalid signature'}, status_code=401)

    data = json.loads(payload)
    event_type = data['event']['eventType']
    document_id = data['document']['documentId']

    if event_type == 'Completed':
        await handle_completed(document_id)
    elif event_type == 'SendFailed':
        logging.error(f"Send failed for {document_id}: {data['document']}")
    elif event_type == 'Declined':
        logging.info(f"Declined: {document_id}")

    return {'status': 'OK'}

async def handle_completed(document_id: str) -> None:
    # Download signed PDF
    pdf = document_api.download_document(document_id)
    with open(f'signed/{document_id}.pdf', 'wb') as f:
        f.write(pdf)

    # Download audit trail
    audit = document_api.download_audit_log(document_id)
    with open(f'signed/{document_id}-audit.pdf', 'wb') as f:
        f.write(audit)

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

---

## Migration Checklist

### Phase 1: Setup
- [ ] Create BoldSign Sandbox account (https://account.boldsign.com/signup?planId=1076)
- [ ] Replace pip package: `pip install boldsign`
- [ ] Replace authentication: Detect DocuSign auth method. If accessToken or Bearer is used then use BoldSign OAuth. Do not switch to API Key unless it is explicitly mentioned.
- [ ] Update base URL configuration
- [ ] Remove all `account_id` parameters from API calls

### Phase 2: Core Document Workflow
- [ ] Migrate send document flow (EnvelopeDefinition → SendForSign)
- [ ] Convert tabs to form fields (see Tab → FormField mapping)
- [ ] Convert recipients to signers (Signer/Reviewer → DocumentSigner with SignerType)
- [ ] Migrate file handling (Base64 → file object)
- [ ] Implement webhook handler for `Sent`/`SendFailed` events (async send)
- [ ] Migrate embedded signing (CreateRecipientView → GetEmbeddedSignLink)

### Phase 3: Advanced Features
- [ ] Migrate templates (TemplateRole.role_name → Roles.role_index)
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
- ❌ Not setting `enable_signing_order = True` for sequential signing
- ❌ Using Docusign `role_name` — BoldSign uses integer `role_index`
- ❌ Not verifying webhook HMAC signatures
- ❌ Polling for document status instead of using webhooks
- ❌ Passing `account_id` to BoldSign API calls
- ❌ Assuming template creation is synchronous — listen for `TemplateCreated` webhook
- ❌ Not handling `Expired` events — Docusign has no equivalent, but BoldSign documents can expire

---

## Reference Links

**BoldSign:**
- Python SDK: https://developers.boldsign.com/sdks/python-sdk/
- PyPI: https://pypi.org/project/boldsign/
- GitHub: https://github.com/boldsign/boldsign-python-sdk
- API Reference: https://developers.boldsign.com/
- Webhooks: https://developers.boldsign.com/webhooks/
- Sandbox Signup: https://account.boldsign.com/signup?planId=1076

**Docusign (for reference during migration):**
- REST API Reference: https://developers.docusign.com/docs/esign-rest-api/reference/
- Python SDK: https://github.com/docusign/docusign-esign-python-client
- Tabs Reference: https://developers.docusign.com/docs/esign-rest-api/esign101/concepts/tabs/
- Connect Webhooks: https://developers.docusign.com/platform/webhooks/connect/
