# Docusign to BoldSign API Migration Skill

A migration skill for converting existing Docusign eSignature integrations to BoldSign eSignature using .NET / C#.

This repository packages the `BoldSign/` skill folder with the root `SKILL.md` entrypoint, which defines production‑ready rules and mappings for migrating Docusign SDK usage to BoldSign SDK usage.

---

## What this repository includes

```text
docusign-to-boldsign-api-migration/
├── README.md
├── LICENSE.txt
├── gitignore.txt
└── BoldSign/
    └── SKILL.md
```

---

## What the skill helps with

Based on the migration guidance defined in `SKILL.md`, this skill helps with:

- Authentication migration from Docusign to BoldSign
- Envelope to Document mapping
- Recipient and routing order migration
- Tab to FormField mapping
- Sending documents for signature
- Embedded signing workflow migration
- Template-based signing workflows
- Status and lifecycle mapping
- Downloading signed documents and audit logs
- Listing and searching documents
- Reminder, revoke, and delete operations
- Webhook migration from Docusign Connect to BoldSign Webhooks

The skill focuses exclusively on migration by replacing Docusign SDK constructs with equivalent BoldSign SDK constructs while preserving existing application logic and workflows.

---

## Installation

1. Download or clone this repository.
2. Copy the `BoldSign/` folder into your current Code Studio or AI agent skills workspace.
3. Provide existing Docusign .NET source code and prompt the agent to perform migration.

---

## Example prompts

- `Migrate Docusign .NET SDK usage to BoldSign .NET SDK usage and generate an equivalent BoldSign source file.`
- `Convert Docusign embedded signing workflow to BoldSign embedded signing.`
- `Translate Docusign template-based envelope flow to BoldSign templates.`
- `Replace Docusign SDK usage with BoldSign SDK equivalents in this C# implementation.`

---

## Repository structure

- `BoldSign/SKILL.md` contains the complete migration rules, mappings, and behavioral guidance.

---

## Notes

- This skill is intended for migration scenarios where Docusign code already exists.
- Generated BoldSign code should be reviewed, integrated, and tested end-to-end.
- Webhook-based status handling is preferred over polling after migration.

---

## Related resources

- BoldSign API documentation: https://developers.boldsign.com
- BoldSign website: https://boldsign.com

---

## License

This project is licensed under the MIT License. See the `LICENSE.txt` file for details.

