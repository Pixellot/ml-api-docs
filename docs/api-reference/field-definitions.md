# Field Definitions

## Root Level Fields

### eventId
- **Type**: String
- **Format**: 24-character hexadecimal
- **Description**: Unique identifier for the basketball game event
- **Example**: `"689cafb95e51ceb1b3440e53"`

### hlsUrl
- **Type**: String (URL)
- **Description**: HTTP Live Streaming URL for the complete game footage
- **Example**: `"https://cdn.example.com/tenant/eventId/venue_hls/hd_hls/hd_hls.m3u8"`

### sport
- **Type**: String (Enum)
- **Values**: `"basketball"`
- **Description**: Sport type identifier
- **Note**: Currently only basketball is supported

### schemaVersion
- **Type**: String
- **Format**: Semantic version (x.y.z)
- **Description**: API schema version for compatibility tracking
- **Example**: `"1.0.0"`

### schemaUrl
- **Type**: String (URL)
- **Description**: URL to the complete JSON schema definition
- **Example**: `"https://api.pixellot.tv/schemas/player-highlights/v1.0.0"`

### processedAt
- **Type**: String (ISO 8601 DateTime)
- **Description**: Timestamp when the ML processing completed
- **Example**: `"2025-08-17T07:14:53.154476Z"`

### players
- **Type**: Object
- **Description**: Container for all player highlight data
- **Key Format**: `{jerseyNumber}_{colorHex}`

## Player Object Fields

### jerseyColor
- **Type**: String
- **Format**: Hex color code with # prefix
- **Description**: Primary color of the player's jersey
- **Example**: `"#ffffff"`, `"#ff0000"`

### jerseyNumber
- **Type**: Integer
- **Range**: 0-99
- **Description**: Player's jersey number
- **Example**: `11`, `23`

### highlights
- **Type**: Array of Objects
- **Description**: Collection of highlight segments for the player

## Highlight Object Fields

### startTime
- **Type**: Number
- **Unit**: Seconds
- **Description**: Beginning timestamp of the highlight segment
- **Example**: `383`, `120.5`

### endTime
- **Type**: Number
- **Unit**: Seconds
- **Description**: End timestamp of the highlight segment
- **Example**: `394`, `125.2`

### type
- **Type**: String (Enum)
- **Values**: `"shot"`, `"assist"`
- **Description**: Classification of the basketball action
- **Definitions**:
  - `shot`: Player successfully made a basket
  - `assist`: Player made a pass that directly led to a made basket
