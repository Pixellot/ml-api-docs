# ML Processing with JSON Output Type

## Overview

The ML Processing with JSON output type is a specialized basketball event processing service that generates structured JSON files containing player highlight timing data instead of individual video clip files. This service is designed for MBE (Match Broadcast Engine) integration and provides precise timestamp information for highlight moments detected through AI analysis.

## Key Benefits

- **Structured Data Output**: Receive comprehensive JSON files with player highlights instead of video clips
- **Real-time Processing**: Get notifications within seconds of ML analysis completion
- **Basketball-Focused**: Optimized specifically for basketball event analysis
- **MBE Integration**: Purpose-built for Match Broadcast Engine systems
- **Precise Timestamps**: Accurate start/end times for each detected highlight

## How It Works

**Processing Flow**

1. Event with JSON output type is submitted for ML processing
2. System validates event configuration and required files
3. ML analysis processes the basketball event to detect player highlights
4. Structured JSON file is generated with player highlight data
5. JSON file is uploaded to customer's S3 bucket
6. Customer receives webhook notification with processing results

## Getting Started

### Prerequisites

- Basketball events with both HD and Panoramic recordings available
- Customer S3 bucket configured in MBE tenant settings
- Webhook endpoint configured to receive processing notifications
- AI Player Highlights extension applied to event configuration

### Event Configuration

To enable ML processing with JSON output, apply the "AI player highlights - Data" extension to your basketball events. This extension is predefined and requires no additional configuration.

For detailed instructions on how to apply the extension to events, see the [Partner API - Adding AI Player Highlights Extension](partner-api-ml-extensions-guide.md) guide.

#### Extension Requirements

| Requirement | Description |
|-------------|-------------|
| **Extension ID** | Use the "AI player highlights - Data" extension ID from your tenant |
| **Production Type** | Event must have `productionType: "basketball"` |
| **Video Requirements** | Both HD and Panoramic recordings must be available |

## Processing Flow Details

### 1. Event Submission
When an event with JSON output type is submitted:
- System validates event configuration and file availability
- Publishes ML processing request to message queue
- Sends **Start Processing** webhook notification

### 2. ML Analysis
The ML service processes the basketball event to:
- Detect player highlight moments using AI analysis
- Identify players by jersey number and team color
- Generate precise start/end timestamps for each highlight
- Classify highlight types (shots, assists, etc.)

### 3. JSON Generation & Upload
Upon successful ML analysis:
- Generates structured JSON file with player highlights data
- Uploads file to customer's S3 bucket at: `events/{eventId}/playerHighlights.json`
- Updates event record with JSON URL and completion status

### 4. Customer Notification
System sends **Success** or **Failed** webhook notification with processing results

## Webhook Notifications

The system sends webhook notifications at three key stages of processing:

### Start Processing Notification

**Webhook Type**: `EventMlBreakdownUpdate`  
**Trigger**: When ML processing begins

```json
{
  "messageType": "EventMlBreakdownUpdate",
  "eventMlBreakdownUpdate": {
    "eventId": "507f1f77bcf86cd799439011",
    "tenantName": "your-tenant",
    "status": "processing",
    "jobCreatedAt": "2024-01-01T12:00:00.000Z"
  }
}
```

### Success Notification

**Webhook Type**: `EventMlBreakdownUpdate`  
**Trigger**: When JSON processing completes successfully

```json
{
  "messageType": "EventMlBreakdownUpdate",
  "eventMlBreakdownUpdate": {
    "eventId": "507f1f77bcf86cd799439011",
    "tenantName": "your-tenant",
    "status": "completed",
    "fileUrl": "https://s3.amazonaws.com/bucket/events/507f1f77bcf86cd799439011/playerHighlights.json",
    "jobCompletedAt": "2024-01-01T12:05:30.000Z"
  }
}
```

### Failed Notification

**Webhook Type**: `EventMlBreakdownUpdate`  
**Trigger**: When processing encounters an error

```json
{
  "messageType": "EventMlBreakdownUpdate",
  "eventMlBreakdownUpdate": {
    "eventId": "507f1f77bcf86cd799439011",
    "tenantName": "your-tenant",
    "status": "failed",
    "reason": "No player highlights detected in processing results",
    "jobFailedAt": "2024-01-01T12:03:15.000Z"
  }
}
```

## JSON File Structure

The generated JSON file contains structured player highlight data organized by team and jersey number.

### Root Schema

```json
{
  "eventId": "string",
  "eventName": "string", 
  "hlsUrl": "string",
  "sportType": "basketball",
  "schemaVersion": "1.0",
  "processedAt": "2024-01-01T12:00:00.000Z",
  "players": {
    "{teamColor}_{jerseyNumber}": {
      "jerseyNumber": "number",
      "teamColor": "string",
      "highlights": [
        {
          "startTime": "number (seconds)",
          "endTime": "number (seconds)", 
          "duration": "number (seconds)",
          "type": "string (shot/assist/etc)",
          "confidence": "number (0-1)"
        }
      ]
    }
  }
}
```

### Example JSON Structure

```json
{
  "eventId": "507f1f77bcf86cd799439011",
  "eventName": "Lakers vs Warriors",
  "hlsUrl": "https://stream.example.com/event.m3u8",
  "sportType": "basketball",
  "schemaVersion": "1.0", 
  "processedAt": "2024-01-01T12:05:30.000Z",
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
        },
        {
          "startTime": 245.2,
          "endTime": 251.7,
          "duration": 6.5,
          "type": "assist",
          "confidence": 0.88
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

## API Endpoints

### Player Highlights JSON Notification

**Endpoint**: `POST /api/events/{eventId}/playerHighlights/json`  
**Purpose**: Receives ML processing results from orchestrator  
**Authentication**: ML Orchestrator Token required

**Request Schema**:
```json
{
  "processedAt": "number (timestamp)",
  "type": "playerHighlights", 
  "format": "json",
  "status": "completed|failed",
  "jsonUrl": "string (required for completed)",
  "reason": "string (required for failed)"
}
```

## Error Handling & Failure Scenarios

### Common Failure Reasons

- **"No UGC bucket configured for upload"**: Missing S3 bucket configuration in tenant settings
- **"Storage upload failed: {error}"**: S3 upload permission or connectivity issues  
- **"ML model inference failed: {error}"**: AI processing errors or model failures
- **"JSON generation failed: {exception}"**: Data formatting or serialization errors
- **"No player highlights detected"**: No highlights found in video analysis results
- **"Event validation failed - missing required files"**: Missing HD/Pano recordings or streams

### Retry Logic

The system implements comprehensive retry mechanisms:

- **Event Validation**: 4 retry attempts with increasing delays (1, 2, 5, 10 minutes)
- **Publisher Connections**: 3 attempts with exponential backoff
- **S3 Operations**: Automatic role assumption with fallback logic
- **Webhook Delivery**: Configurable timeout with error logging

## Integration Requirements

### S3 Bucket Configuration

- Customer must provide UGC bucket details in MBE tenant settings
- Proper IAM permissions for file upload access
- Cross-region access configuration if required
- Bucket versioning recommended for data recovery

### Webhook Endpoint Setup

- Customer must configure MBE notification endpoint URL
- Endpoint must accept `EventMlBreakdownUpdate` message type
- HTTPS endpoint with valid SSL certificate required
- Response time under 10 seconds to prevent timeouts
- Proper authentication and error handling recommended

### JSON Schema Compatibility

- Generated JSON follows versioned schema for forward compatibility
- Schema version included in each file for validation purposes
- Backward compatibility maintained across schema updates
- Custom parsers should handle unknown fields gracefully

## Implementation Best Practices

### 1. Handle Webhook Notifications Efficiently

- Respond to webhooks quickly (under 3 seconds) with HTTP 200
- Process JSON file downloads asynchronously after acknowledging receipt
- Implement idempotency to handle duplicate notifications gracefully

### 2. Implement Robust File Processing

- Use retry logic with exponential backoff for S3 file downloads
- Validate JSON schema before processing highlight data
- Handle missing or malformed data gracefully
- Implement circuit breakers to prevent cascading failures

### 3. Ensure Data Security

- Verify webhook authenticity if signature verification is implemented
- Use secure connections (HTTPS) for all API communications
- Implement proper access controls for downloaded JSON files
- Log all processing activities for audit and troubleshooting

### 4. Optimize for Performance

- Cache frequently accessed highlight data locally
- Implement parallel processing for multiple events
- Use appropriate data structures for fast highlight lookups
- Consider database indexing strategies for timestamp queries

## Monitoring & Observability

### Event Status Tracking

Events are tracked through these status states:
- **processing**: ML analysis in progress
- **completed**: JSON successfully generated and uploaded
- **failed**: Processing encountered error

### Key Metrics to Monitor

- Processing duration and success rate
- JSON file size and player count statistics
- S3 upload success/failure rates
- Webhook delivery success rates
- Error categorization and frequency

### Logging & Debugging

- Structured logging with correlation IDs for request tracing
- Error categorization for troubleshooting patterns
- Performance metrics for optimization opportunities
- Audit trails for compliance and debugging

## Troubleshooting Guide

| Issue | Possible Causes | Solutions |
|-------|----------------|-----------|
| Missing notifications | • Webhook endpoint not accessible<br>• Network connectivity issues | • Verify webhook URL accessibility<br>• Check firewall and DNS settings |
| Processing failures | • Missing required files<br>• Invalid event configuration | • Verify HD/Pano recordings exist<br>• Check AI extension is applied to event |
| JSON download issues | • S3 permissions problems<br>• Network connectivity | • Verify S3 bucket access<br>• Check IAM role permissions |
| No highlights detected | • Poor video quality<br>• Non-basketball content | • Verify video quality and sport type<br>• Check camera angles and lighting |

## Service Limits

- **Processing Timeout**: 30 minutes maximum per event
- **File Size Limits**: JSON files typically under 1MB
- **Retry Attempts**: Maximum 4 retries for validation failures
- **Webhook Timeout**: 10 seconds response time limit
- **Concurrent Processing**: Limited by ML model capacity

## Support Resources

For technical support with ML processing integration:

1. **Check Event Status**: Use the event API to verify processing status
2. **Review Webhook Logs**: Check your webhook endpoint logs for delivery issues
3. **Validate Configuration**: Ensure AI Player Highlights extension is applied to event
4. **Contact Support**: Email ml-api-support@pixellot.com with:
   - Event ID and tenant name
   - Timestamp of the issue
   - Webhook notification details
   - Any error messages received

This service provides a reliable, scalable solution for basketball highlight analysis with comprehensive error handling and customer notification at every stage of processing.
