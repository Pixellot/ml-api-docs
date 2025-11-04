# Pixellot ML Breakdown Schema

JSON Schema specification for ML-generated basketball player highlights data.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ML Player Highlights API Schema",
  "description": "Schema for ML-generated player highlights data from basketball game",
  "type": "object",
  "required": [
    "eventId",
    "hlsUrl", 
    "sport",
    "schemaVersion",
    "schemaUrl",
    "processedAt",
    "players"
  ],
  "properties": {
    "eventId": {
      "type": "string",
      "description": "Unique event identifier (24-character hex)",
      "pattern": "^[a-f0-9]{24}$"
    },
    "hlsUrl": {
      "type": "string",
      "format": "uri",
      "description": "Main HLS stream URL for the event"
    },
    "sport": {
      "type": "string",
      "enum": ["basketball"],
      "description": "Sport type (currently supports basketball)"
    },
    "schemaVersion": {
      "type": "string",
      "description": "Schema version for data format compatibility",
      "pattern": "^v\\d+\\.\\d+\\.\\d+$"
    },
    "schemaUrl": {
      "type": "string",
      "format": "uri",
      "description": "URL to the JSON schema definition"
    },
    "processedAt": {
      "type": "string",
      "format": "date-time",
      "description": "Processing completion timestamp in ISO 8601 format"
    },
    "players": {
      "type": "object",
      "description": "Player data keyed by {jerseyNumber}_{colorHex}",
      "patternProperties": {
        "^\\d{1,2}_[a-fA-F0-9]{6}$": {
          "type": "object",
          "required": [
            "jerseyColor",
            "jerseyNumber",
            "highlights"
          ],
          "properties": {
            "jerseyColor": {
              "type": "string",
              "pattern": "^#[a-fA-F0-9]{6}$",
              "description": "Hex color code (e.g., '#ffffff')"
            },
            "jerseyNumber": {
              "type": "integer",
              "minimum": 0,
              "description": "Player jersey number (0-99)"
            },
            "highlights": {
              "type": "array",
              "description": "Array of highlight segments for the player",
              "items": {
                "type": "object",
                "required": [
                  "startTime",
                  "endTime", 
                  "type"
                ],
                "properties": {
                  "startTime": {
                    "type": "number",
                    "minimum": 0,
                    "description": "Segment start timestamp in seconds"
                  },
                  "endTime": {
                    "type": "number",
                    "minimum": 0,
                    "description": "Segment end timestamp in seconds"
                  },
                  "type": {
                    "type": "string",
                    "enum": ["shot", "assist"],
                    "description": "Action type (shot or assist)"
                  }
                }
              }
            }
          }
        }
      },
      "additionalProperties": false
    },
    "unknownPlayers": {
      "type": "array",
      "description": "Shot/assist detections where player identification failed",
      "items": {
        "type": "object",
        "required": [
          "startTime",
          "endTime",
          "color_hex",
          "playerN"
        ],
        "properties": {
          "startTime": {
            "type": "number",
            "minimum": 0,
            "description": "Segment start timestamp in seconds"
          },
          "endTime": {
            "type": "number",
            "minimum": 0,
            "description": "Segment end timestamp in seconds"
          },
          "color_hex": {
            "type": "string",
            "description": "Team color in hex format (empty string if unknown)"
          },
          "playerN": {
            "type": "integer",
            "description": "Player number (-1 for unknown)"
          }
        }
      }
    }
  },
  "additionalProperties": false
}
```
