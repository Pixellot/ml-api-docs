# Basketball Player Highlights

## Overview

Get AI-generated basketball player highlights as structured JSON data instead of video clips. Perfect for creating your own highlight reels, analytics dashboards, or integrating with broadcast systems.

## What You Get

- **Player Highlights**: AI detects shots, assists, and key plays
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
- HD and panoramic camera recordings

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
  "eventName": "Lakers vs Warriors",
  "players": {
    "home_23": {
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
    "away_30": {
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
- `players`: Organized by `{team}_{jerseyNumber}` format
- `startTime`/`endTime`: Highlight timestamps in seconds
- `type`: Action type (`shot`, `assist`, etc.)

## Common Issues

**No highlights detected?**
- This happens when the AI doesn't find clear player actions
- Make sure your event has both HD and panoramic recordings
- Check that the event is properly configured as `productionType: "basketball"`

**Processing takes too long?**
- Processing typically completes within 10-30 minutes after video is available
- Longer videos or poor quality footage may take additional time

**Missing JSON file?**
- Files are uploaded to your configured storage bucket
- Check that your storage configuration is correct
- Files remain available for 30 days after processing

## Need Help?

Contact **ml-api-support@pixellot.com** with:
- Event ID
   - Timestamp of the issue
- Any error messages from webhooks

For more details on setting up notifications, see the [Basketball Highlight Notifications guide](ml-breakdown-subscription-guide.md).
