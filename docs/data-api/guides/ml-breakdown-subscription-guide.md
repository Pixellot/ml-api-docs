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
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"processing\"}"
}
```

### 2. Processing Completed ✅
```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"https://cdn.example.com/highlights.json\",\"status\":\"completed\"}"
}
```

### 3. Processing Failed ❌
```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"failed\",\"reason\":\"No highlights detected\"}"
}
```

**Key Fields:**
- `eventId`: Your event identifier
- `fileUrl`: Download link for JSON data (when completed)
- `status`: `processing`, `completed`, or `failed`
- `reason`: Error details (when failed)

## What the JSON Data Looks Like

When processing completes, download the JSON file to get player highlights:

```json
{
  "eventId": "68ac3315e9dde9b6a61d71d7",
  "eventName": "Lakers vs Warriors",
  "players": {
    "home_23": {
      "jerseyNumber": 23,
      "highlights": [
        {
          "startTime": 125.5,
          "endTime": 132.8,
          "type": "shot"
        }
      ]
    }
  }
}
```

Each highlight shows:
- **startTime/endTime**: When the highlight occurs (in seconds)
- **type**: What happened (`shot`, `assist`, etc.)
- **jerseyNumber**: Which player made the play

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
  const { eventId, status, fileUrl, reason } = data;
  
  if (status === 'completed' && fileUrl) {
    // Download and process the JSON file
    downloadHighlights(eventId, fileUrl);
  } else if (status === 'failed') {
    console.log(`Processing failed for ${eventId}: ${reason}`);
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

**File download fails?**  
- Files are available for 30 days after processing
- Add retry logic with a few seconds delay

**No highlights detected?**
- This happens when no clear player actions are found in the video
- Check that your event has both HD and panoramic camera views

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

## Need Help?

Contact **ml-api-support@pixellot.com** with:
- Your subscription ID
- Event ID (if applicable)  
- Webhook endpoint URL
- Any error messages
