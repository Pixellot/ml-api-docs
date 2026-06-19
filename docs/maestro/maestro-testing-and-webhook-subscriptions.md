# Maestro Testing & Webhook Subscriptions

## Maestro API — Getting Started: Authentication & Publishing a Game

This guide walks you through the two essential steps to start using the Maestro API:

1. Obtain an access token (authentication)
2. Publish a game event to start highlight processing

### Prerequisites

Before you begin, make sure you have:

- A valid Maestro account (email and password provided by your Pixellot account manager)
- The base URL for the Maestro API (e.g., `https://maestro.stage.pixellot.tv`)
- A tool to make HTTP requests — examples below use `curl`, but any REST client works (Postman, Insomnia, etc.)

---

## Step 1 — Fetch an Access Token

All Maestro API endpoints (except login) require a valid access token. You obtain this token by logging in with your credentials.

**Endpoint**

```
POST /auth/login
```

**Request Headers**

| Header | Value |
|--------|-------|
| Content-Type | application/json |

**Request Body**

```json
{
  "email": "your-email@example.com",
  "password": "yourPassword123"
}
```

**Field Reference**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | ✅ Yes | Your account email address. Must be a valid email format. |
| password | string | ✅ Yes | Your account password. Must be at least 8 characters long. |

**Example Request**

```bash
curl -X POST https://maestro.stage.pixellot.tv/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "your-email@example.com",
    "password": "yourPassword123"
  }'
```

**Example Response**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user-abc-123",
    "email": "your-email@example.com",
    "firstName": "Jane",
    "lastName": "Doe",
    "role": "editor"
  }
}
```

**Response Field Reference**

| Field | Type | Description |
|-------|------|-------------|
| accessToken | string | JWT access token — include this in all subsequent API calls. It is short-lived (expires in ~1 hour). |
| refreshToken | string | JWT refresh token — use this to obtain a new access token when the current one expires, without re-entering your password. Store this securely. |
| user.id | string | Unique identifier for your account. |
| user.email | string | Your account email. |
| user.firstName | string | Your first name. |
| user.lastName | string | Your last name. |
| user.role | string | Your permission level (e.g., `admin`, `viewer`). Controls what API operations you are allowed to perform. |

> ⚠️ **Security note:** Treat both tokens like passwords. Do not log them, share them, or commit them to source control. The access token expires automatically. If your access token expires, use the refresh token flow (see below) instead of re-entering credentials.

---

## Step 1b — Refreshing an Expired Access Token (Optional)

When your access token expires, you can get a new one using your refresh token — no need to log in again.

**Endpoint**

```
POST /auth/refresh
```

**Request Body**

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Example Response**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...(new token)...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...(new token)..."
}
```

Both a new access token and a new refresh token are returned. Update your stored tokens accordingly.

---

## Step 2 — Publish a Game Event

Once you have an access token, you can publish a game event. This tells Maestro to start processing video chunks and generating highlights for the game.

**Endpoint**

```
POST /game-event/publish
```

**Request Headers**

| Header | Value |
|--------|-------|
| Content-Type | application/json |
| Authorization | Bearer `<your-access-token-here>` |

Replace `<your-access-token-here>` with the `accessToken` you received in Step 1.

**Request Body**

```json
{
  "eventId": "evt_1234567890",
  "tenant": "sportsEngine",
  "sportType": "basketball",
  "hlsUrl": "https://cdn.example.com/live/stream.m3u8",
  "s3Info": {
    "bucketName": "your-highlights-bucket",
    "region": "us-east-1",
    "distributionUrl": "https://d1abc123xyz.cloudfront.net",
    "savePath": "highlights/your-tenant-id/evt_1234567890",
    "roleToAssume": "arn:aws:iam::123456789012:role/highlight-processor-role"
  }
}
```

**Field Reference — Top Level**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| eventId | string | ✅ Yes | A unique identifier for this game event. 24 Alpha Numeric Characters. Used to track the event throughout the system — make sure it is unique per game and consistent if you reference the event later. |
| tenant | string | ✅ Yes | Your tenant (organization) identifier, as provisioned by Pixellot. Scopes the event to your account and ensures isolation from other customers. Example: `acme-sports`. |
| sportType | string | ✅ Yes | Type of sport being played. Used to select the correct ML model and processing configuration. Example values: `soccer`, `basketball`, `american-football`. Use the value provided by Pixellot onboarding. |
| hlsUrl | string | ✅ Yes | Full URL of the HLS manifest (`.m3u8`) for the live or recorded game video. Maestro uses this to download and process video chunks. Example: `https://cdn.example.com/live/game123/index.m3u8`. |
| s3Info | object | ✅ Yes | AWS S3 storage configuration for where processed highlights should be saved. Required if you want Maestro to store output clips to your own S3 bucket. If omitted, default storage settings will be used. |

**Field Reference — `s3Info` Object**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| bucketName | string | ✅ Yes | Name of your AWS S3 bucket where highlight clips will be saved. Example: `my-highlights-bucket`. |
| region | string | ✅ Yes | AWS region where your S3 bucket is located. Example: `us-east-1`, `eu-west-1`. |
| distributionUrl | string | ✅ Yes | Base public URL used to access files in your bucket — typically a CloudFront CDN distribution URL. This is the URL prefix end users will use to play clips. |
| savePath | string | ✅ Yes | Folder path inside your S3 bucket where highlights will be stored. Should be specific enough to organize files by tenant and event. |
| roleToAssume | string | ❌ No | ARN of an AWS IAM role that Maestro should assume in order to write to your S3 bucket. Required if your bucket uses cross-account access. |

**Example Request**

```bash
curl -X POST https://maestro.stage.pixellot.tv/game-event/publish \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d '{
    "eventId": "evt_1234567890",
    "tenant": "sportsEngine",
    "sportType": "basketball",
    "hlsUrl": "https://cdn.example.com/live/game123/index.m3u8",
    "s3Info": {
      "bucketName": "sports-engine",
      "region": "us-east-1",
      "distributionUrl": "https://d1abc123xyz.cloudfront.net",
      "savePath": "/sportsEngine/evt_1234567890/venue_hls",
      "roleToAssume": "arn:aws:iam::123456789012:role/maestro-writer-role"
    }
  }'
```

**Example Response**

A successful publish returns HTTP `201 Created` with no response body.

```
HTTP/1.1 201 Created
```

This means Maestro has accepted the event and queued it for processing. There is no further payload — tracking the processing progress is done via webhooks or the event status endpoint (see related pages).

**Error Reference**

| HTTP Status | Meaning | What to check |
|-------------|---------|---------------|
| 400 Bad Request | Request body is missing required fields or has invalid values | Check that all required fields are present and correctly formatted. The response body will include a `message` array describing which fields failed validation. |
| 401 Unauthorized | Missing or invalid access token | Ensure the `Authorization: Bearer <token>` header is present and the token hasn't expired. If expired, refresh it (see Step 1b). |
| 403 Forbidden | Account does not have permission | Contact your Pixellot account manager to verify your role and permissions. |
| 500 Internal Server Error | Unexpected error on the server | Retry after a short wait. If the error persists, contact Pixellot support with the request ID if available. |

---

## EventMlBreakdownUpdate Subscription Guide

### Overview

Get real-time notifications when basketball player highlights are ready. This webhook subscription tells you when AI processing starts, completes, or fails, so you can immediately access the generated JSON data.

**What You'll Get**

- Instant notifications when processing completes
- Direct links to download JSON highlight data
- Error alerts if processing fails
- Simple webhook integration

---

### Quick Setup

#### 1. Get Your API Token

```bash
curl -X POST https://api.pixellot.tv/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USERNAME", "password": "YOUR_PASSWORD"}'
```

Save the returned token — you'll need it for the next step.

> **Note:** These are not your Maestro credentials for testing, but your Pixellot API credentials.

#### 2. Create Your Subscription

```bash
curl -X POST https://api.pixellot.tv/v1/monitoring/subscriptions \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messageType": "EventMlBreakdownUpdate",
    "url": "https://your-webhook-endpoint.com/basketball-highlights",
    "emails": "alerts@example.com",
    "tenant": "YOUR_TENANT_ID",
    "secret": "your-shared-secret",
    "type": "external"
  }'
```

**Field Reference:**

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `messageType` | Yes | string (enum) | The notification type. Use `"EventMlBreakdownUpdate"` for basketball highlight updates. |
| `url` | Yes | string | Your HTTPS webhook endpoint that will receive the notification. |
| `emails` | Yes | string | Comma-separated email list (single string, not an array). Example: `"alerts@example.com,ops@example.com"`. |
| `tenant` | No | string | Your tenant identifier (provided by Pixellot). |
| `secret` | No | string | Shared secret for signing/authenticating the webhook payload. |
| `type` | No | string (enum) | Subscription scope — `"internal"` or `"external"`. Omit if unspecified. |

**Minimal body (required fields only):**

```json
{
  "messageType": "EventMlBreakdownUpdate",
  "url": "https://your-webhook-endpoint.com/basketball-highlights",
  "emails": "alerts@example.com"
}
```

---

### Webhook Notifications

You'll receive three types of notifications:

#### 1. Processing Started

```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"processing\",\"jobCreatedAt\":\"2024-01-15T10:30:00Z\"}"
}
```

#### 2. Processing Completed ✅

```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"fileUrl\":\"https://cdn.example.com/highlights.json\",\"status\":\"completed\",\"jobCompletedAt\":\"2024-01-15T10:45:30Z\"}"
}
```

#### 3. Processing Failed ❌

```json
{
  "payload": "{\"eventId\":\"68ac3315e9dde9b6a61d71d7\",\"status\":\"failed\",\"reason\":\"Event validation failed because of missing required files\"}"
}
```

**Key Fields:**

| Field | Present in | Description |
|-------|-----------|-------------|
| `eventId` | All notifications | Your event identifier |
| `status` | All notifications | `processing`, `completed`, or `failed` |
| `jobCreatedAt` | `processing` only | ISO 8601 timestamp when processing job was created |
| `fileUrl` | `completed` only | Download link for JSON highlight data |
| `jobCompletedAt` | `completed` only | ISO 8601 timestamp when processing job finished |
| `reason` | `failed` only | Error details |

---

### JSON Data Structure

The full schema for the basketball player highlights JSON output is defined in **Pixellot ML Breakdown Schema**. When processing completes, you'll receive a JSON file that follows this specification.

---

### Simple Webhook Handler

Your endpoint needs to:

1. Return HTTP 200 quickly (under 10 seconds)
2. Use HTTPS
3. Handle the three notification types

**Basic Example (Node.js)**

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
  // Fetch and process the highlights JSON
}
```

---

### Managing Subscriptions

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
