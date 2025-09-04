# Basketball Highlight Notifications

## Overview

Get real-time notifications when basketball player highlights are ready. This webhook subscription tells you when AI processing starts, completes, or fails, so you can immediately access the generated JSON data.

## What You'll Get

- Instant notifications when processing completes
- Direct links to download JSON highlight data
- Error alerts if processing fails
- Simple webhook integration

## Quick Setup

### 1. Get Your API Token

```bash
curl -X POST https://api.pixellot.tv/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USERNAME", "password": "YOUR_PASSWORD"}'
```

Save the returned `token` - you'll need it for the next step.

### 2. Create Your Subscription

```bash
curl -X POST https://api.pixellot.tv/v1/monitoring/subscriptions \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messageType": "EventMlBreakdownUpdate",
    "tenant": "YOUR_TENANT_ID",
    "url": "https://your-webhook-endpoint.com/basketball-highlights"
  }'
```

**Required Parameters:**
- `messageType`: Always use `"EventMlBreakdownUpdate"`
- `tenant`: Your tenant ID (provided by Pixellot)  
- `url`: Your HTTPS webhook endpoint

## Webhook Notifications

You'll receive three types of notifications:

### 1. Processing Started
```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"processing\",\"jobCreatedAt\":\"2024-01-15T10:30:00Z\"}"
}
```

### 2. Processing Completed ✅
```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"https://cdn.example.com/highlights.json\",\"status\":\"completed\",\"jobCompletedAt\":\"2024-01-15T10:45:30Z\"}"
}
```

### 3. Processing Failed ❌
```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"failed\",\"reason\":\"Event validation failed because of missing required files\"}"
}
```

**Key Fields:**
- `eventId`: Your event identifier (always present)
- `status`: `processing`, `completed`, or `failed` (always present)
- `jobCreatedAt`: ISO 8601 timestamp when processing job was created (only in "processing" notifications)
- `fileUrl`: Download link for JSON data (only in "completed" notifications)
- `jobCompletedAt`: ISO 8601 timestamp when processing job finished (only in "completed" notifications)
- `reason`: Error details (only in "failed" notifications)

## JSON Data Structure

The full schema for the basketball player highlights JSON output is defined in [Pixellot ML Breakdown Schema](../specifications/pixellot-ml-breakdown-schema.md).

When processing completes, you'll receive a JSON file that follows this specification.

## Simple Webhook Handler

Your endpoint needs to:
1. Return HTTP 200 quickly (under 10 seconds)
2. Use HTTPS
3. Handle the three notification types

### Basic Example (Node.js)

```javascript
app.post('/basketball-highlights', (req, res) => {
  // Always respond quickly first
  res.status(200).send('OK');
  
  // Parse the notification
  const data = JSON.parse(req.body.payload);
  const { eventId, status, fileUrl, reason, jobCreatedAt, jobCompletedAt } = data;
  
  if (status === 'completed' && fileUrl) {
    // Download and process the JSON file
    downloadHighlights(eventId, fileUrl);
  } else if (status === 'failed') {
    // Failed notifications only contain eventId, status, and reason
    console.log(`Processing failed for ${eventId}: ${reason}`);
  } else if (status === 'processing') {
    // Processing started - jobCreatedAt is available
    console.log(`Processing started for ${eventId} at ${jobCreatedAt}`);
  }
});

async function downloadHighlights(eventId, fileUrl) {
  const response = await fetch(fileUrl);
  const highlights = await response.json();
  
  // Process your highlights data
  console.log(`Got ${Object.keys(highlights.players).length} players for event ${eventId}`);
}
```

## Common Issues

**Not receiving notifications?**
- Check your webhook endpoint is publicly accessible via HTTPS
- Verify your subscription is active: `GET /v1/monitoring/subscriptions`


## Managing Your Subscription

```bash
# List subscriptions
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.pixellot.tv/v1/monitoring/subscriptions

# Update webhook URL  
curl -X PUT -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://new-endpoint.com/webhook"}' \
  https://api.pixellot.tv/v1/monitoring/subscriptions/SUBSCRIPTION_ID

# Delete subscription
curl -X DELETE -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.pixellot.tv/v1/monitoring/subscriptions/SUBSCRIPTION_ID
```
