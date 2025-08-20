# Response Format

## Base Structure

```json
{
  "eventId": "689cafb95e51ceb1b3440e53",
  "hlsUrl": "https://cdn.example.com/tenant/eventId/venue_hls/hd_hls/hd_hls.m3u8",
  "sport": "basketball",
  "schemaVersion": "1.0.0", 
  "schemaUrl": "https://api.pixellot.tv/schemas/player-highlights/v1.0.0",
  "processedAt": "2025-08-17T07:14:53.154476Z",
  "players": {
    "11_ffffff": {
      "jerseyColor": "#ffffff",
      "jerseyNumber": 11,
      "highlights": [
        {
          "startTime": 383,
          "endTime": 394, 
          "type": "shot"
        }
      ]
    }
  }
}
```

## Key Characteristics

- **JSON Format**: All responses are in JSON format
- **UTF-8 Encoding**: Standard UTF-8 text encoding
- **Consistent Structure**: All responses follow the same base structure
- **Nullable Fields**: Some fields may be null depending on processing results

## Player Key Format

Players are keyed using the format: `{jerseyNumber}_{colorHex}`

**Examples:**
- `11_ffffff`: Player #11 with white jersey
- `23_ff0000`: Player #23 with red jersey
- `7_0000ff`: Player #7 with blue jersey

## Timestamp Format

All timestamps use:
- **Unit**: Seconds (integer or float)
- **Reference**: Relative to video start time
- **Precision**: Sub-second accuracy when available
