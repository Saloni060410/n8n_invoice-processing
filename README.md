# Invoice Processing

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?logo=n8n&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-Inbox%20Automation-EA4335?logo=gmail&logoColor=white)
![Gemini](https://img.shields.io/badge/Google%20Gemini-Vision%20OCR-4285F4?logo=google&logoColor=white)
![Zoho Books](https://img.shields.io/badge/Zoho%20Books-Billing%20Workflow-1D4ED8?logo=zoho&logoColor=white)
![Workflow](https://img.shields.io/badge/Workflow-Invoice%20Processing-0F766E)

Automated invoice processing workflow built in n8n to watch a Gmail inbox, extract invoice data from email attachments with Gemini Vision, create the vendor and bill in Zoho Books, and send a confirmation email back to the sender.

## What This Workflow Does

This workflow is designed for invoice emails that arrive as attachments in Gmail. It runs every 5 minutes, looks for messages in the inbox with attachments, and processes only emails whose subject contains the word `invoice`.

Supported attachment types:

- PDF invoices
- Image-based invoices

If the attachment is not a PDF or image, the workflow sends a notification email to the configured internal address.

## End-to-End Flow

1. Gmail Trigger polls the inbox for messages with attachments.
2. An IF check keeps only messages whose subject contains `invoice`.
3. The workflow downloads the attachment and validates the file type.
4. The attachment is converted to base64.
5. Gemini Vision is called with a structured prompt to extract invoice fields as XML.
6. The XML is parsed into JSON.
7. Required fields are validated.
8. The invoice is mapped to Zoho Books bill format.
9. The workflow checks whether the vendor already exists in Zoho Books.
10. If needed, the vendor is created.
11. A bill is created in Zoho Books.
12. If the bill is returned as `draft`, it is submitted and moved to open.
13. A confirmation email is sent back to the original sender.

## Main Integrations

- Gmail for inbound invoice collection and outbound notifications
- Google Gemini Vision API for OCR and structured extraction
- Zoho Books API for vendor lookup, vendor creation, and bill creation

## Repository Contents

- [invoice.json](invoice.json): the n8n workflow export
- [README.md](README.md): project documentation
- [workflow_view.png](workflow_view.png): a quick view of how the workflow looks
- [Saloni-self_assessment.pdf](Saloni-self_assessment.pdf): what problems I faced, how I solved it and the final output

## Prerequisites

Before importing and running the workflow, make sure you have:

- An active n8n instance
- A connected Gmail OAuth2 credential in n8n
- A connected Zoho OAuth2 credential in n8n
- A Google Gemini API key with access to the Gemini 2.5 Flash model
- A Zoho Books organization ID
- Permission to create vendors and bills in the target Zoho Books organization

## Setup

1. Import `invoice.json` into n8n.
2. Open the Gmail nodes and confirm they use your Gmail OAuth2 credential.
3. Open the Zoho HTTP Request nodes and confirm they use your Zoho OAuth2 credential.
4. Replace the hardcoded Gemini API key in the workflow with your own secure configuration.
5. Review the hardcoded Zoho organization ID and account ID values in the code nodes and HTTP requests.
6. Update the internal notification email address used for invalid attachments or extraction failures.
7. Activate the workflow.

## Configuration Notes

The workflow currently assumes the following:

- The Gmail subject line contains `invoice`
- The invoice arrives as an attachment, not a link
- Gemini returns XML in the exact schema requested by the prompt
- Zoho Books vendor search is performed using the extracted vendor name
- New bills are created in the configured Zoho Books organization

Important implementation details:

- The Gemini call uses the `gemini-2.5-flash` vision endpoint.
- The extraction prompt requires these fields at minimum: vendor name, invoice number, total amount.
- Line items are mapped into Zoho bill line items.
- If the bill is created in draft status, the workflow submits it automatically.

## Workflow Stages

### Stage 1: Mail Extraction

This stage watches Gmail, filters invoice emails, and downloads the attachment for processing.

### Stage 2: Gemini Connection and Text Extraction

The attachment is converted to base64 and sent to Gemini Vision for invoice field extraction.

### Stage 3: Bill Creation

The extracted invoice data is validated, mapped to Zoho Books schema, and used to create the vendor and bill.

### Stage 4: Confirmation of Mail

After the bill is created and submitted if required, the workflow sends a confirmation email to the original sender.

## Error Handling

The workflow includes two main failure paths:

- Invalid attachment type: sends an internal email saying the attachment is not PDF or image format.
- Missing required invoice fields: sends an internal email saying extraction is incomplete.

## Output

When the workflow succeeds, the sender receives an email containing:

- Invoice number
- Invoice date
- Amount due
- Due date
- ERP reference / bill ID from Zoho Books

## Security and Maintenance

- Do not commit real API keys or OAuth secrets into version control.
- Move sensitive values into n8n credentials or environment variables where possible.
- Review any hardcoded organization, account, and notification values before production use.
- Test the workflow with sample invoices before enabling it on a live inbox.

## Troubleshooting

- If emails are not being processed, confirm the Gmail trigger is active and the subject contains `invoice`.
- If extraction fails, verify that the attachment is readable by Gemini and that the PDF or image quality is sufficient.
- If Zoho bill creation fails, confirm the Zoho OAuth credential has the correct permissions and the organization ID is valid.
- If confirmation emails are not sent, check the Gmail OAuth credential and the sender address parsing logic.

## Suggested Next Step

After importing, run a test invoice through the flow and confirm that the vendor lookup, bill creation, and confirmation email all complete successfully.

