# Enable Basketball AI Highlights

## Overview

Add AI-powered player highlight detection to your basketball events. This extension automatically analyzes game footage and generates JSON files with player highlight data.

## What It Does

- **Detects Key Plays**: Automatically finds made shots and assists
- **Identifies Players**: Tracks actions by jersey color and number
- **Provides Timestamps**: Precise start/end times for each highlight
- **Generates JSON**: Structured data you can integrate with your systems

## Quick Setup

### 1. Get Your API Token

```bash
curl -X POST https://api.pixellot.tv/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USERNAME", "password": "YOUR_PASSWORD"}'
```

Save the returned `token` for the next steps.

### 2. Find the Extension ID

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.pixellot.tv/v1/tenants/YOUR_TENANT/extensions
```

Look for the extension named **"AI player highlights - Data"** and copy its `_id` value.

### 3. Add Extension to Events

**For new events:**
```bash
curl -X POST https://api.pixellot.tv/v1/events \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "eventName": "Lakers vs Warriors",
    "start$date": "2024-12-15T18:00:00.000Z",
    "end$date": "2024-12-15T20:00:00.000Z",
    "productionType": "basketball",
    "venue": {"_id": "YOUR_VENUE_ID"},
    "extensions": {
      "extensionsIdsToApply": ["YOUR_EXTENSION_ID"]
    }
  }'
```

**For existing events:**
```bash
curl -X PUT https://api.pixellot.tv/v1/events/EVENT_ID \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "extensions": {
      "extensionsIdsToApply": ["YOUR_EXTENSION_ID"]
    }
  }'
```

**Important:** Set `productionType: "basketball"` for AI processing to work.

## Verify It Worked

Check that the extension was added:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.pixellot.tv/v1/events/EVENT_ID
```

Look for `"extensionsIdsToApply": ["YOUR_EXTENSION_ID"]` in the response.

## What Happens Next

1. **Automatic Processing**: When your event has video recordings, AI processing starts automatically
2. **Get Notified**: Set up [webhook notifications](ml-breakdown-subscription-guide.md) to know when processing completes
3. **Download Data**: Access the generated JSON file with player highlights

## Common Issues

**Extension not found?**
- Make sure you're using the correct tenant name in the API call
- Contact support if the extension isn't available for your account

**Extension won't apply?**
- Double-check the extension ID from the list call
- Verify your API token has the right permissions

**No AI processing happening?**
- Ensure `productionType` is set to `"basketball"`

**API authentication failing?**
- Your token may have expired - get a new one
- Verify your username and password are correct

## Related Guides

- [Basketball Player Highlights](ml-processing-json-output-guide.md) - What the AI processing does
- [Basketball Highlight Notifications](ml-breakdown-subscription-guide.md) - Get notified when processing completes
