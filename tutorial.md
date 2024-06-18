### Tutorial: Using Prefect Webhooks to Consume Events from Azure Event Grid 

This tutorial will guide you through setting up Prefect webhooks to consume events from Azure Event Grid, specifically for events related to a file being dropped into Azure Blob Storage.

### Prerequisites 

- Azure Storage account with a blob container.

- Azure Service Bus namespace and topic.

- Prefect Cloud account.

### Step 1: Set Up Prefect Webhook 
 
1. **Create a Webhook in Prefect Cloud:**  
  - Log in to your Prefect Cloud account.
 
  - Navigate to the menu on the left, click `Configuration`, then `Webhooks`.
 
  - Click `Create Webhook`.
 
  - Name your webhook (e.g., `AzureBlobEventWebhook`).
 
  - Add a description if desired.
 
  - Select `Dynamic` for the webhook type.
 
  - Define the webhook template using Jinja2:


```json
{
    "event": "{{body[0].eventType}}",
    "payload": {
        "topic": "{{body[0].topic}}",
        "subject": "{{body[0].subject}}",
        "eventType": "{{body[0].eventType}}",
        "eventTime": "{{body[0].eventTime}}",
        "id": "{{body[0].id}}",
        "data": {
            "api": "{{body[0].data.api}}",
            "clientRequestId": "{{body[0].data.clientRequestId}}",
            "requestId": "{{body[0].data.requestId}}",
            "eTag": "{{body[0].data.eTag}}",
            "contentType": "{{body[0].data.contentType}}",
            "contentLength": "{{body[0].data.contentLength}}",
            "blobType": "{{body[0].data.blobType}}",
            "url": "{{body[0].data.url}}",
            "sequencer": "{{body[0].data.sequencer}}",
            "storageDiagnostics": {
                "batchId": "{{body[0].data.storageDiagnostics.batchId}}"
            }
        },
        "dataVersion": "",
        "metadataVersion": "1"
    },
    "resource": {
        "prefect.resource.id": "{{ body[0].data.url }}"
    }
}
```
 
  - Click `Create`.
 
2. **Get Prefect Webhook URL:** 
  - After creating the webhook, copy the unique URL provided by Prefect Cloud.

### Step 2: Set Up Azure Event Grid 
 
1. **Enable Event Grid Integration:** 
  - Navigate to your Azure storage account in the Azure portal.
 
  - Go to `Events` under `Settings`.
 
  - Click on `+ Event Subscription`.
 
  - Name your subscription (e.g., `BlobCreatedSubscription`).
 
  - For `Event Schema`, select `Event Grid Schema`.
 
  - For `Event Type`, select `Blob created`.
 
  - For `Endpoint type`, select `Webhook`.

  - Paste the Prefect webhook URL copied earlier.

  - Save the changes.

### Step 3: Verify the Webhook 
 
1. **Verify the Webhook:** 
  - When you save the Event Grid subscription with the Prefect webhook URL, Azure Event Grid sends a verification request to the webhook.

  - Go to your Prefect Cloud dashboard to view the event payload received.

  - Look for the verification URL in the payload.

  - Copy the verification URL and paste it into your browser navigation bar where you are logged into Azure.

  - This will complete the verification process, allowing the webhook configuration to save properly on the Azure side.

### Step 4: Configure Prefect Automation 
 
1. **Create Automation in Prefect Cloud:**  
  - In Prefect Cloud, navigate to `Automations`.
 
  - Create a new automation to trigger flows based on the event received.
 
  - Use the following trigger JSON:


```json
{
  "type": "event",
  "match": {
    "prefect.resource.id": "https://kevinstest.blob.core.windows.net/data/*"
  },
  "match_related": {},
  "after": [],
  "expect": [
    "Microsoft.Storage.BlobCreated"
  ],
  "for_each": [],
  "posture": "Reactive",
  "threshold": 1,
  "within": 0
}
```

### Conclusion 

This tutorial guides you through setting up Prefect webhooks and automations to consume events from Azure Event Grid, including the necessary verification step. Following these steps will help you create a reactive system that responds to file uploads in Azure Blob Storage, allowing you to automate workflows and processes seamlessly.
