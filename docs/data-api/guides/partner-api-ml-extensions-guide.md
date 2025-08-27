# Partner API - Adding AI Player Highlights Extension

## Overview

The "AI player highlights - Data" extension is a predefined extension that enables AI-powered player highlight detection and data analytics for basketball events. Since it's predefined in the system, you only need the extension ID to activate all ML processing capabilities - no additional configuration is required.

This guide shows you how to use the Partner API (PAPI) to add the AI extension to your events, enabling automatic generation of JSON files with player highlight data.

## Key Benefits

- **Predefined Configuration**: Extension comes pre-configured with optimal settings for basketball
- **Simple Integration**: Only requires the extension ID to activate
- **Automatic Processing**: ML analysis starts automatically when events are processed
- **JSON Output**: Generates structured player highlight data files
- **No Manual Setup**: All AI models and processing pipelines are managed automatically

## Authentication

### Step 1: Login to Partner API

First, authenticate with the Pixellot Partner API to obtain your access token:

```http
POST https://api.pixellot.tv/v1/login
Content-Type: application/json

{
  "username": "your_username", 
  "password": "your_password"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "user": {
    "id": "user_id",
    "username": "your_username",
    "tenant": "your_tenant"
  }
}
```

### Step 2: Use Authentication Token

Include the returned token in all subsequent API requests:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Finding the Extension ID

### Get Available Extensions

Retrieve the list of available extensions for your tenant:

```http
GET https://api.pixellot.tv/v1/tenants/{tenantName}/extensions
Authorization: Bearer your_api_token
```

**Response:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439055",
    "name": "AI player highlights - Data",
    "description": "AI-powered player highlight detection with data analytics",
    "status": "active",
    "type": "ml_processing",
    "sportTypes": ["basketball"],
    "createdAt": "2024-01-01T12:00:00.000Z"
  },
  {
    "_id": "507f1f77bcf86cd799439056",
    "name": "Other Extension",
    "description": "Different extension",
    "status": "active"
  }
]
```

### Identify the ML Extension

Look for the extension with:
- **Name**: `"AI player highlights - Data"`
- **Status**: `"active"`
- **Type**: `"ml_processing"` (if available)
- **Sport Types**: Should include `"basketball"`

Note the `_id` field - this is what you'll use to apply the extension to events.

## Adding Extension to Events

### Method 1: Create New Event with Extension

Create a new basketball event with the AI extension already applied:

```http
POST https://api.pixellot.tv/v1/events
Authorization: Bearer your_api_token
Content-Type: application/json

{
  "eventName": "Basketball Championship Final",
  "start$date": "2024-12-15T18:00:00.000Z", 
  "end$date": "2024-12-15T20:00:00.000Z",
  "status": "active",
  "venue": {
    "_id": "your_venue_id"
  },
  "productionType": "basketball",
  "permission": "admin",
  "extensions": {
    "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
  }
}
```

**Response:**
```json
{
  "_id": "event_id_12345",
  "eventName": "Basketball Championship Final",
  "start$date": "2024-12-15T18:00:00.000Z",
  "end$date": "2024-12-15T20:00:00.000Z",
  "status": "active",
  "productionType": "basketball",
  "extensions": {
    "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
  },
  "createdAt": "2024-12-01T10:00:00.000Z"
}
```

### Method 2: Add Extension to Existing Event

Add the AI extension to an event that already exists:

```http
PUT https://api.pixellot.tv/v1/events/{eventId}
Authorization: Bearer your_api_token
Content-Type: application/json

{
  "extensions": {
    "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
  }
}
```

**Important Fields:**

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `eventName` | string | Name of the basketball event | Yes (for new events) |
| `start$date` | string | Event start time in ISO 8601 format | Yes (for new events) |
| `end$date` | string | Event end time in ISO 8601 format | Yes (for new events) |
| `productionType` | string | Must be `"basketball"` for ML processing | Yes (for new events) |
| `venue._id` | string | Venue identifier | Yes (for new events) |
| `extensions.extensionsIdsToApply` | array | Array of extension IDs to apply | Yes |

## Command Line Examples

### cURL - Create Event with Extension

```bash
curl -X POST "https://api.pixellot.tv/v1/events" \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json" \
  -d '{
    "eventName": "Basketball Championship Final",
    "start$date": "2024-12-15T18:00:00.000Z",
    "end$date": "2024-12-15T20:00:00.000Z",
    "status": "active",
    "venue": {
      "_id": "your_venue_id"
    },
    "productionType": "basketball",
    "permission": "admin",
    "extensions": {
      "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
    }
  }'
```

### cURL - Add Extension to Existing Event

```bash
curl -X PUT "https://api.pixellot.tv/v1/events/event_id_12345" \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json" \
  -d '{
    "extensions": {
      "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
    }
  }'
```

### PowerShell Example

```powershell
$headers = @{
    "Authorization" = "Bearer your_api_token"
    "Content-Type" = "application/json"
}

$body = @{
    eventName = "Basketball Championship Final"
    "start`$date" = "2024-12-15T18:00:00.000Z"
    "end`$date" = "2024-12-15T20:00:00.000Z"
    status = "active"
    venue = @{ _id = "your_venue_id" }
    productionType = "basketball"
    permission = "admin"
    extensions = @{
        extensionsIdsToApply = @("507f1f77bcf86cd799439055")
    }
} | ConvertTo-Json -Depth 3

Invoke-RestMethod -Uri "https://api.pixellot.tv/v1/events" -Method POST -Headers $headers -Body $body
```

## Verification and Monitoring

### Verify Extension Application

After adding the extension, verify it was applied correctly:

```http
GET https://api.pixellot.tv/v1/events/{eventId}
Authorization: Bearer your_api_token
```

Check that the response includes:
```json
{
  "_id": "event_id_12345",
  "extensions": {
    "extensionsIdsToApply": ["507f1f77bcf86cd799439055"]
  }
}
```

### Monitor Processing Status

Once the event is processed, you can monitor ML processing through:

1. **EventMlBreakdownUpdate Subscriptions**: Set up webhook notifications (see [EventMlBreakdownUpdate Subscription Guide](ml-breakdown-subscription-guide.md))
2. **Event Status Checks**: Poll the event endpoint to check processing status
3. **Generated Files**: Look for JSON files in your configured S3 bucket

## Integration with ML Processing

### Automatic Activation

When you add the "AI player highlights - Data" extension to a basketball event:

1. **Extension Recognition**: System automatically detects the ML extension
2. **Processing Queue**: Event is queued for ML processing when video is available
3. **AI Analysis**: Basketball-specific AI models analyze the video for player highlights
4. **JSON Generation**: System creates structured JSON files with highlight data
5. **File Upload**: JSON files are uploaded to your configured S3 bucket
6. **Notifications**: Webhook notifications are sent (if subscriptions are configured)

### What Gets Processed

The extension enables processing of:
- **Player Highlights**: Shot attempts, successful shots, assists
- **Timestamp Data**: Precise start/end times for each highlight
- **Player Identification**: Jersey numbers and team assignments
- **Confidence Scores**: AI confidence levels for each detected highlight
- **Game Context**: Event metadata and video stream references

## Best Practices

### 1. Event Configuration

- **Production Type**: Always use `"basketball"` for basketball events
- **Timing**: Set realistic start/end dates for your events
- **Venue**: Ensure venue ID is valid and associated with your tenant
- **Status**: Use `"active"` for live events, `"archived"` for completed events

### 2. Extension Management

- **Single Extension**: Only apply the AI extension once per event
- **Timing**: Add extension before or during event creation for best results
- **Verification**: Always verify extension was applied successfully

### 3. Error Handling

```javascript
// Example error handling in JavaScript
async function addMlExtension(eventId, extensionId) {
  try {
    const response = await fetch(`https://api.pixellot.tv/v1/events/${eventId}`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${apiToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        extensions: {
          extensionsIdsToApply: [extensionId]
        }
      })
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const result = await response.json();
    console.log('Extension added successfully:', result);
    return result;

  } catch (error) {
    console.error('Failed to add ML extension:', error);
    throw error;
  }
}
```

### 4. Monitoring and Logging

- **Log API Calls**: Keep records of extension applications for debugging
- **Monitor Processing**: Set up webhook subscriptions for real-time updates
- **Error Tracking**: Implement proper error handling and logging
- **Status Checks**: Periodically verify extension status on critical events

## Troubleshooting

### Common Issues

| Issue | Possible Causes | Solutions |
|-------|----------------|-----------|
| **Extension not found** | • Incorrect tenant name<br>• Extension not available | • Verify tenant name in API call<br>• Contact support to enable extension |
| **Extension application failed** | • Invalid extension ID<br>• Insufficient permissions | • Verify extension ID from list call<br>• Check API token permissions |
| **No ML processing** | • Wrong production type<br>• Extension not active | • Ensure productionType is "basketball"<br>• Verify extension status is "active" |
| **Authentication errors** | • Expired token<br>• Invalid credentials | • Refresh authentication token<br>• Verify username/password |

### Debugging Steps

1. **Verify Authentication**: Test login endpoint and token validity
2. **Check Extension List**: Confirm extension exists and is active for your tenant
3. **Validate Event Data**: Ensure all required fields are present and correct
4. **Test API Calls**: Use tools like Postman or curl to test requests manually
5. **Review Logs**: Check API response messages for specific error details

### Getting Help

For issues with Partner API integration:

1. **API Documentation**: Review complete PAPI documentation for additional endpoints
2. **Extension Support**: Contact support if extension is not available for your tenant
3. **Technical Support**: Email ml-api-support@pixellot.com with:
   - Tenant name and username
   - Extension ID being used
   - Event ID (if applicable)
   - Full API request and response
   - Any error messages received

## Related Documentation

- [ML Processing with JSON Output Type](ml-processing-json-output-guide.md) - Complete guide to ML processing workflow
- [EventMlBreakdownUpdate Subscription Guide](ml-breakdown-subscription-guide.md) - Setting up webhook notifications
- [Schema Documentation](../specifications/pixellot-dataapi-statstics-schema.md) - API schema and data structures

## Summary

The "AI player highlights - Data" extension provides a simple way to enable ML processing for basketball events:

1. **Authenticate** with the Partner API
2. **Find** the extension ID for your tenant
3. **Apply** the extension to basketball events
4. **Monitor** processing through webhooks or status checks

That's it! The extension is predefined and automatically configured, so simply including the extension ID enables all AI player highlight detection and data analytics features for your basketball events.
