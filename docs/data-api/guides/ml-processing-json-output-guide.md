# Basketball Player Highlights

## Overview

Get AI-generated basketball player highlights as structured JSON data. Perfect for creating your own highlight reels, analytics dashboards, or integrating with broadcast systems.

## What You Get

- **Player Highlights**: AI detects made shots and assists
- **Precise Timing**: Exact start/end timestamps for each highlight
- **JSON Format**: Structured data that's easy to integrate
- **Real-time Processing**: Get notified when analysis completes

## How It Works

1. **Enable the Feature**: Add the AI Player Highlights extension to your basketball events
2. **Automatic Processing**: Our AI analyzes the game video when it's available  
3. **JSON Generation**: Structured highlight data is created and uploaded to your storage
4. **Get Notified**: Receive a webhook notification with a link to download the JSON

## Getting Started

### What You Need

- Basketball events with video recordings
- The "AI player highlights - Data" extension enabled on your events
- A webhook endpoint to receive notifications (optional but recommended)

### Enable AI Highlights

Add the AI extension to your basketball events using the [Partner API guide](partner-api-ml-extensions-guide.md).

The extension automatically processes events that have:
- `productionType: "basketball"`

## Notifications

You'll receive webhook notifications when:
- **Processing starts**: AI analysis begins
- **Processing completes**: JSON file is ready for download
- **Processing fails**: Something went wrong

See the [Basketball Highlight Notifications guide](ml-breakdown-subscription-guide.md) for webhook setup.

## JSON Data Structure

When processing completes, you'll get a JSON file with this structure:

### Example JSON File

```json
{
  "eventId": "507f1f77bcf86cd799439011",
  "hlsUrl": "https://cdn.example.com/tenant/eventId/venue_hls/hd_hls/hd_hls.m3u8",
  "sport": "basketball",
  "schemaVersion": "v1.0.0",
  "schemaUrl": "https://raw.githubusercontent.com/Pixellot/ml-api-docs/refs/tags/v1.0.0/schema.json",
  "processedAt": "2024-01-15T14:30:45Z",
  "players": {
    "23_ffffff": {
      "jerseyColor": "#ffffff",
      "jerseyNumber": 23,
      "highlights": [
        {
          "startTime": 125.5,
          "endTime": 132.8,
          "type": "shot"
        },
        {
          "startTime": 245.2,
          "endTime": 251.7,
          "type": "assist"
        }
      ]
    },
    "30_ff0000": {
      "jerseyColor": "#ff0000",
      "jerseyNumber": 30,
      "highlights": [
        {
          "startTime": 178.3,
          "endTime": 185.1,
          "type": "shot"
        }
      ]
    }
  }
}
```

**Key Fields:**
- `eventId`: Your basketball event identifier
- `hlsUrl`: Main HLS stream URL for the event
- `sport`: Sport type (currently basketball)
- `schemaVersion`: API schema version
- `processedAt`: When processing completed (ISO 8601 timestamp)
- `players`: Organized by `{jerseyNumber}_{colorHex}` format containing:
  - `jerseyColor`: Player's jersey color (hex code)
  - `jerseyNumber`: Player's jersey number
  - `highlights`: Array of highlight segments with:
    - `startTime`/`endTime`: Highlight timestamps in seconds
    - `type`: Action type (`shot` or `assist`)

## Common Issues

**No highlights detected?**
- This happens when the AI doesn't find clear player actions
- Check that the event is properly configured as `productionType: "basketball"`

**Processing takes too long?**
- Processing completes up to 4 hours after the video is available

For more details on setting up notifications, see the [Basketball Highlight Notifications guide](ml-breakdown-subscription-guide.md).
