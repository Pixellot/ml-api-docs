# EventMlBreakdownUpdate Subscription Guide

## Overview

The EventMlBreakdownUpdate subscription provides real-time notifications for ML processing of basketball events that generate JSON files with player highlight data instead of video clips. This subscription service allows you to receive instant notifications at each stage of the ML processing pipeline, ensuring you can track processing status and retrieve results as soon as they become available.

## Key Benefits

- **Real-time Notifications**: Receive instant updates when ML processing starts, completes, or fails
- **Comprehensive Status Tracking**: Monitor the complete lifecycle of ML processing jobs
- **Direct File Access**: Get immediate access to generated JSON files with player highlight data
- **Error Handling**: Receive detailed error information when processing fails
- **Reliable Delivery**: Built-in retry mechanisms ensure notification delivery

## Authentication & Setup

### Obtaining Your Authentication Token

First, obtain your authentication token by logging into the Pixellot API:

```http
POST https://api.pixellot.tv/v1/auth/login
Content-Type: application/json

{
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD"
}
```

**Response:**
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600
}
```

Store the returned token securely as you'll need it for all subsequent API calls.

## Creating a Subscription

### Create EventMlBreakdownUpdate Subscription

To subscribe to ML breakdown notifications, send a POST request to the monitoring subscriptions endpoint:

```http
POST /v1/monitoring/subscriptions
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json

{
    "messageType": "EventMlBreakdownUpdate",
    "tenant": "YOUR_TENANT_ID", 
    "url": "https://your-webhook-endpoint.com/ml-breakdown"
}
```

### Request Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `messageType` | string | Must be `"EventMlBreakdownUpdate"` | Yes |
| `tenant` | string | Your tenant identifier (provided by Pixellot) | Yes |
| `url` | string | Your HTTPS webhook endpoint URL | Yes |

### Response

```json
{
    "id": 9457,
    "messageType": "EventMlBreakdownUpdate",
    "tenant": "your-tenant-id",
    "url": "https://your-webhook-endpoint.com/ml-breakdown",
    "status": "active",
    "createdAt": "2024-01-01T12:00:00.000Z"
}
```

## Webhook Notifications

You'll receive notifications at three distinct stages of ML processing. Each notification contains the same basic structure but with different status values and relevant data.

### 1. Processing Started

**Trigger**: Sent when ML analysis begins  
**Status**: `processing`

```json
{
  "url": "https://your-webhook-endpoint.com/ml-breakdown",
  "hash": "",
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"\",\"status\":\"processing\",\"jobCreatedAt\":\"2025-08-26T14:50:00.000Z\",\"jobCompletedAt\":\"\",\"reason\":\"\"}",
  "subscriptionId": 9457,
  "messageType": "EventMlBreakdownUpdate"
}
```

### 2. Processing Completed (Success)

**Trigger**: Sent when JSON file is successfully generated and uploaded  
**Status**: `completed`

```json
{
  "url": "https://your-webhook-endpoint.com/ml-breakdown",
  "hash": "",
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"https://d1zc5juudnw5vp.cloudfront.net/glo-tenant/68ac3315e9dde9b6a61d71d7/68ac3315e9dde9b6a61d71d7_player_highlights.json\",\"status\":\"completed\",\"jobCreatedAt\":\"\",\"jobCompletedAt\":\"2025-08-26T14:55:12.859Z\",\"reason\":\"\"}",
  "subscriptionId": 9457,
  "messageType": "EventMlBreakdownUpdate"
}
```

### 3. Processing Failed

**Trigger**: Sent when processing encounters an error  
**Status**: `failed`

```json
{
  "url": "https://your-webhook-endpoint.com/ml-breakdown", 
  "hash": "",
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"\",\"status\":\"failed\",\"jobCreatedAt\":\"\",\"jobCompletedAt\":\"\",\"reason\":\"ML model inference failed: timeout\"}",
  "subscriptionId": 9457,
  "messageType": "EventMlBreakdownUpdate"
}
```

## Payload Structure

The JSON payload (nested within the `payload` string) contains the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | string | Unique event identifier |
| `fileUrl` | string | URL to download JSON file (only when status=completed) |
| `status` | string | Current processing status: `processing` \| `completed` \| `failed` |
| `jobCreatedAt` | string | ISO timestamp when processing started |
| `jobCompletedAt` | string | ISO timestamp when processing finished |
| `reason` | string | Error description (only when status=failed) |

## JSON File Content

When the status is `"completed"`, you can download the player highlight data from the `fileUrl`. The JSON file contains structured basketball highlight data:

```json
{
  "eventId": "68ac3315e9dde9b6a61d71d7",
  "eventName": "Game Name",
  "hlsUrl": "https://example.com/event.m3u8",
  "sportType": "basketball", 
  "schemaVersion": "1.0",
  "processedAt": "2025-08-26T14:55:12.859Z",
  "players": {
    "home_23": {
      "jerseyNumber": 23,
      "teamColor": "home",
      "highlights": [
        {
          "startTime": 125.5,
          "endTime": 132.8,
          "duration": 7.3,
          "type": "shot",
          "confidence": 0.95
        }
      ]
    },
    "away_30": {
      "jerseyNumber": 30,
      "teamColor": "away",
      "highlights": [
        {
          "startTime": 178.3,
          "endTime": 185.1,
          "duration": 6.8,
          "type": "shot",
          "confidence": 0.92
        }
      ]
    }
  }
}
```

### File Structure Details

- **Event Metadata**: Basic information about the basketball event
- **Player Organization**: Players grouped by `{teamColor}_{jerseyNumber}` keys
- **Highlight Data**: Each highlight includes precise timestamps, duration, type, and confidence score
- **Schema Version**: Version information for compatibility checking

## Webhook Implementation

### Response Requirements

Your webhook endpoint must meet these requirements:

- **Response Time**: Return HTTP 200-299 status code within 10 seconds
- **Processing**: Process data asynchronously after responding to prevent timeouts
- **HTTPS**: Endpoint must use secure HTTPS protocol
- **Availability**: Endpoint should be publicly accessible

### Example Handler (Node.js)

```javascript
app.post('/ml-breakdown', (req, res) => {
  try {
    // Parse the nested JSON payload
    const { eventId, fileUrl, status, reason } = JSON.parse(req.body.payload);
    
    // Respond immediately to acknowledge receipt
    res.status(200).send('OK');
    
    // Process asynchronously to avoid timeout
    setImmediate(() => {
      if (status === 'completed' && fileUrl) {
        processPlayerHighlights(eventId, fileUrl);
      } else if (status === 'failed') {
        handleProcessingFailure(eventId, reason);
      } else if (status === 'processing') {
        updateProcessingStatus(eventId);
      }
    });
    
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(400).send('Bad Request');
  }
});

async function processPlayerHighlights(eventId, fileUrl) {
  try {
    // Download the JSON file
    const response = await fetch(fileUrl);
    const highlightData = await response.json();
    
    // Validate schema version
    if (highlightData.schemaVersion !== '1.0') {
      console.warn(`Unsupported schema version: ${highlightData.schemaVersion}`);
    }
    
    // Process player highlights
    for (const [playerKey, playerData] of Object.entries(highlightData.players)) {
      console.log(`Processing ${playerData.highlights.length} highlights for player ${playerData.jerseyNumber}`);
      // Your highlight processing logic here
    }
    
  } catch (error) {
    console.error(`Failed to process highlights for event ${eventId}:`, error);
  }
}

function handleProcessingFailure(eventId, reason) {
  console.error(`ML processing failed for event ${eventId}: ${reason}`);
  // Your error handling logic here
}

function updateProcessingStatus(eventId) {
  console.log(`ML processing started for event ${eventId}`);
  // Your status tracking logic here
}
```

### Example Handler (Python/Flask)

```python
from flask import Flask, request, jsonify
import json
import requests
import threading

app = Flask(__name__)

@app.route('/ml-breakdown', methods=['POST'])
def handle_ml_breakdown():
    try:
        # Parse the nested JSON payload
        payload_data = json.loads(request.json['payload'])
        event_id = payload_data['eventId']
        file_url = payload_data.get('fileUrl', '')
        status = payload_data['status']
        reason = payload_data.get('reason', '')
        
        # Respond immediately
        response = jsonify({'status': 'received'})
        
        # Process asynchronously
        if status == 'completed' and file_url:
            threading.Thread(target=process_player_highlights, args=(event_id, file_url)).start()
        elif status == 'failed':
            threading.Thread(target=handle_processing_failure, args=(event_id, reason)).start()
        elif status == 'processing':
            threading.Thread(target=update_processing_status, args=(event_id,)).start()
        
        return response
        
    except Exception as e:
        print(f'Webhook processing error: {e}')
        return jsonify({'error': 'Bad Request'}), 400

def process_player_highlights(event_id, file_url):
    try:
        # Download the JSON file
        response = requests.get(file_url)
        highlight_data = response.json()
        
        # Validate schema version
        if highlight_data.get('schemaVersion') != '1.0':
            print(f"Unsupported schema version: {highlight_data.get('schemaVersion')}")
        
        # Process player highlights
        for player_key, player_data in highlight_data.get('players', {}).items():
            print(f"Processing {len(player_data['highlights'])} highlights for player {player_data['jerseyNumber']}")
            # Your highlight processing logic here
            
    except Exception as e:
        print(f'Failed to process highlights for event {event_id}: {e}')

def handle_processing_failure(event_id, reason):
    print(f'ML processing failed for event {event_id}: {reason}')
    # Your error handling logic here

def update_processing_status(event_id):
    print(f'ML processing started for event {event_id}')
    # Your status tracking logic here
```

## Managing Subscriptions

You can manage your subscriptions using these API endpoints:

### List All Subscriptions

```http
GET /v1/monitoring/subscriptions
Authorization: Bearer YOUR_TOKEN
```

### Update Subscription

```http
PUT /v1/monitoring/subscriptions/{id}
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json

{
    "url": "https://your-new-webhook-endpoint.com/ml-breakdown"
}
```

### Delete Subscription

```http
DELETE /v1/monitoring/subscriptions/{id}
Authorization: Bearer YOUR_TOKEN
```

## Best Practices

### 1. Handle All Status Types
Process all three notification types (`processing`, `completed`, `failed`) to maintain complete visibility into the ML processing pipeline.

### 2. Implement Retry Logic
For failed file downloads, implement exponential backoff retry logic:

```javascript
async function downloadWithRetry(url, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await fetch(url);
            if (response.ok) {
                return await response.json();
            }
            throw new Error(`HTTP ${response.status}`);
        } catch (error) {
            if (attempt === maxRetries) {
                throw error;
            }
            const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}
```

### 3. Store Processing State
Track event processing status in your system to handle duplicate notifications and maintain processing history.

### 4. Validate JSON Schema
Always check the `schemaVersion` before processing highlight data to ensure compatibility:

```javascript
function validateSchema(highlightData) {
    const supportedVersions = ['1.0'];
    const version = highlightData.schemaVersion;
    
    if (!supportedVersions.includes(version)) {
        throw new Error(`Unsupported schema version: ${version}`);
    }
}
```

### 5. Quick Response
Always acknowledge webhook notifications within 10 seconds to prevent retries and potential duplicate processing.

### 6. Secure Endpoints
- Use HTTPS for all webhook endpoints
- Implement proper authentication if required
- Validate request signatures if provided
- Log all webhook activities for debugging

## Troubleshooting

### Common Issues and Solutions

| Issue | Possible Causes | Solutions |
|-------|----------------|-----------|
| **Missing notifications** | • Subscription inactive<br>• Webhook endpoint not accessible<br>• Network connectivity issues | • Verify subscription status via API<br>• Check webhook endpoint accessibility<br>• Verify firewall and DNS settings |
| **File download fails** | • Network connectivity problems<br>• File URL expired<br>• Insufficient permissions | • Check network connectivity<br>• Retry download with backoff<br>• Verify file URL validity |
| **No highlights data** | • Event processing found no highlights<br>• Poor video quality<br>• Non-basketball content | • Check event logs for processing details<br>• Verify video quality and sport type<br>• Review camera angles and lighting |
| **Webhook timeouts** | • Slow response processing<br>• Synchronous file downloads<br>• Database blocking operations | • Implement asynchronous processing<br>• Use background job queues<br>• Optimize database queries |
| **Duplicate notifications** | • Network retry mechanisms<br>• Processing delays<br>• System redundancy | • Implement idempotency checks<br>• Track processed event IDs<br>• Use database constraints |

### Debugging Steps

1. **Check Subscription Status**: Verify your subscription is active using the list subscriptions API
2. **Test Webhook Endpoint**: Ensure your endpoint is publicly accessible and returns 200 status codes
3. **Review Logs**: Check your application logs for webhook processing errors
4. **Validate Payload**: Ensure you're correctly parsing the nested JSON payload structure
5. **Monitor Processing Times**: Track how long your webhook responses take to identify timeout issues

### Getting Help

For additional support with EventMlBreakdownUpdate subscriptions:

1. **API Documentation**: Review the complete API documentation for subscription management
2. **Event Logs**: Check event processing logs in the Pixellot dashboard
3. **Network Testing**: Use tools like `curl` or Postman to test your webhook endpoints
4. **Contact Support**: Email ml-api-support@pixellot.com with:
   - Subscription ID
   - Event ID (if applicable)
   - Timestamp of the issue
   - Webhook endpoint URL
   - Any error messages or logs

## Service Limits

- **Maximum Subscriptions**: 5 active subscriptions per tenant per message type
- **Webhook Timeout**: 10 seconds maximum response time
- **Retry Attempts**: Up to 3 retry attempts with exponential backoff
- **File Availability**: JSON files remain accessible for 30 days after generation
- **Rate Limiting**: API calls are subject to standard rate limits (100 requests/minute)

This subscription service provides a reliable way to stay informed about ML processing status and retrieve basketball highlight data as soon as it becomes available.
