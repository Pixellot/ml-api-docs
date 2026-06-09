# Moment Schema

A **moment** represents a single detected sports event (e.g. a shot, a foul, a pass) produced by the ML pipeline or Manual breakdown. Each moment is a JSON document divided into four top-level sections: `control`, `scope`, `stats`, and `visuals`.

---

## `control`

Metadata about the moment record itself — who created it, when, and its position in the processing pipeline.

| Field | Type | Description |
|---|---|---|
| `momentID` | string (ObjectId) | Unique identifier for this moment. |
| `parentMomentID` | string (ObjectId) | ID of the parent moment this one was derived from, if any. |
| `createdAt` | string (ISO 8601) | UTC timestamp of when the moment was created. |
| `createdBy` | string | Identifier of the model or process that generated this moment (e.g. `"ml_model v3.2"`). |
| `momentSequence` | integer | Ordinal position of this moment within the event. Used for ordering moments chronologically. |
| `parentSequence` | integer | Sequence number of the parent moment. |
| `processingTimeSec` | number | Time in seconds it took the pipeline to produce this moment. |
| `reviewStatus` | string | Current review state of the moment. Possible values: `"auto-approved"`, `"pending"`, `"rejected"`. |

---

## `scope`

Defines what the moment is about — the sport, event, timing, and primary actors involved.

| Field | Type | Description                                                                                                                                                               |
|---|---|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sport` | string | Sport this moment belongs to (e.g. `"basketball"`).                                                                                                                       |
| `eventID` | string (ObjectId) | ID of the game/event this moment belongs to.                                                                                                                              |
| `momentType` | string | Type of action detected (e.g. `"shot"`, `"foul"`, `"rebound"`).                                                                                                           |
| `gamePeriod` | integer | Game period (quarter, half, etc.) in which the moment occurred.                                                                                                           |
| `start` | object | Moment start position. Contains `frame` (frame number), `utc` (ISO 8601 timestamp), and `seconds` (elapsed seconds from stream start). Not all time metrics are mandatory |
| `end` | object | Moment end position. Same structure as `start`.                                                                                                                           |
| `mainTeamID` | string (ObjectId) | ID of the primary team associated with this moment.                                                                                                                       |
| `mainTeamColor` | string (hex RGBA) | Primary color of the main team in hex RGBA format (e.g. `"FFFFFFFF"`).                                                                                                    |
| `mainPlayerFID` | string | Federated ID of the primary player involved, in `{teamID}/{jerseyNumber}` format.                                                                                         |
| `confidence` | number [0–1] | Model confidence score for this moment's detection.                                                                                                                       |

---

## `stats`

The statistical attributes captured for this moment, organised by schema version and grouped by team, player, and moment-level attributes.

| Field | Type | Description |
|---|---|---|
| `attributesSchemaVersion` | string | Version of the attributes schema used (e.g. `"1.0"`). Governs which attribute keys are valid. |
| `teamAttributes` | object | Map of stat keys to team IDs. Keys follow the convention `{sport_prefix}_team_{stat}` (e.g. `bb_team_shot`). The value is the ID of the team credited with that stat. |
| `playerAttributes` | object | Map of stat keys to player federated IDs (`{teamID}/{jerseyNumber}`). Keys follow the convention `{sport_prefix}_player_{role}` (e.g. `bb_player_shot`, `bb_player_assist`). |
| `momentAttributes` | object | Map of stat keys to scalar values describing the moment itself. Keys follow the convention `{sport_prefix}_{stat}` (e.g. `bb_shot_result`, `bb_shot_points`). Values can be strings, numbers, or booleans depending on the attribute. |

### Basketball player attribute keys (`bb_player_*`)

| Key | Description |
|---|---|
| `bb_player_shot` | Player who attempted the shot. |
| `bb_player_assist` | Player who assisted the shot. |
| `bb_player_foul_committed` | Player who committed a foul. |
| `bb_player_foul_drawn` | Player who drew the foul. |
| `bb_player_defender` | Primary defending player on the play. |

### Basketball moment attribute keys (`bb_*`)

| Key | Type | Description |
|---|---|---|
| `bb_shot_result` | string | Outcome of the shot: `"made"` or `"missed"`. |
| `bb_shot_points` | integer | Point value of the shot (2 or 3). |
| `bb_shot_type` | string | Shot classification (e.g. `"jumper"`, `"layup"`, `"dunk"`). |
| `bb_foul_type` | string | Type of foul called (e.g. `"personal"`, `"flagrant"`, `"technical"`). |

---

## `visuals`

Defines how the moment should be presented visually, including per-player highlight segments and links to tracking/overlay assets.

| Field | Type | Description |
|---|---|---|
| `segments` | array | List of visual segments, one per relevant player or role. See [Segments](#segments) below. |
| `boundingBoxURL` | string | Storage path to a JSON file containing bounding-box player-tracking data for this moment. |
| `verticalURL` | string | Storage path to a JSON file containing the vertical-view overlay data for this moment. |

### Segments

Each segment defines a clip window around a specific player's involvement in the moment.

| Field | Type | Description |
|---|---|---|
| `statRef` | array of strings | One or more keys from `stats.playerAttributes` that this segment is associated with (e.g. `["playerAttributes.bb_player_shot"]`). |
| `relativeOffset` | object | Start of the clip relative to `scope.start`. Contains `frame` (frame delta) and `seconds` (seconds delta). Negative values indicate the clip starts before the moment's start. |
| `relativeEnd` | object | End of the clip relative to `scope.start`. Same structure as `relativeOffset`. |
| `playerFID` | string | Federated ID (`{teamID}/{jerseyNumber}`) of the player featured in this segment. |
| `teamFID` | string (ObjectId) | ID of the team the featured player belongs to. |
| `confidence` | number [0–1] | Model confidence that this player is correctly identified for this segment. |

---

## Full Example

```json
{
  "control": {
    "momentID": "65cb1dfb0b7a241a9e448600",
    "parentMomentID": "65cb1dfb0b7a241a66648600",
    "createdAt": "2026-03-22T14:32:00Z",
    "createdBy": "ml_model v3.2",
    "momentSequence": 42,
    "parentSequence": 10,
    "processingTimeSec": 12.4,
    "reviewStatus": "auto-approved"
  },
  "scope": {
    "sport": "basketball",
    "eventID": "65cb1dfb0b7a241a9e448600",
    "momentType": "shot",
    "gamePeriod": 2,
    "start": { "frame": 59484, "utc": "2026-03-22T14:32:12Z", "seconds": 1932 },
    "end": { "frame": 59544, "utc": "2026-03-22T14:32:14Z", "seconds": 1934 },
    "mainTeamID": "65cb1dfb0b7a241a9e448639",
    "mainTeamColor": "FFFFFFFF",
    "mainPlayerFID": "65cb1dfb0b7a241a9e448639/J7",
    "confidence": 0.97
  },
  "stats": {
    "attributesSchemaVersion": "1.0",
    "teamAttributes": {
      "bb_team_shot": "65cb1dfb0b7a241a9e448639"
    },
    "playerAttributes": {
      "bb_player_shot": "65cb1dfb0b7a241a9e448639/J7",
      "bb_player_assist": "65cb1dfb0b7a241a9e448639/J4",
      "bb_player_foul_committed": "65cb1dfb0b7a241a9e448640/J12",
      "bb_player_foul_drawn": "65cb1dfb0b7a241a9e448639/J7",
      "bb_player_defender": "65cb1dfb0b7a241a9e448640/J12"
    },
    "momentAttributes": {
      "bb_shot_result": "made",
      "bb_shot_points": 3,
      "bb_shot_type": "jumper",
      "bb_foul_type": "personal"
    }
  },
  "visuals": {
    "segments": [
      {
        "statRef": ["playerAttributes.bb_player_assist"],
        "relativeOffset": { "frame": -66, "seconds": -2.2 },
        "relativeEnd": { "frame": 0, "seconds": 0 },
        "playerFID": "65cb1dfb0b7a241a9e448639/J4",
        "teamFID": "65cb1dfb0b7a241a9e448639",
        "confidence": 0.91
      },
      {
        "statRef": ["playerAttributes.bb_player_shot", "playerAttributes.bb_player_foul_drawn"],
        "relativeOffset": { "frame": 0, "seconds": 0 },
        "relativeEnd": { "frame": 60, "seconds": 2 },
        "playerFID": "65cb1dfb0b7a241a9e448639/J7",
        "teamFID": "65cb1dfb0b7a241a9e448639",
        "confidence": 0.97
      },
      {
        "statRef": ["playerAttributes.bb_player_foul_committed", "playerAttributes.bb_player_defender"],
        "relativeOffset": { "frame": -30, "seconds": -1.0 },
        "relativeEnd": { "frame": 60, "seconds": 2 },
        "playerFID": "65cb1dfb0b7a241a9e448640/J12",
        "teamFID": "65cb1dfb0b7a241a9e448640",
        "confidence": 0.84
      }
    ],
    "boundingBoxURL": "sportsEngine/65cb1dfb0b7a241a9e448600/pixellot_ml/highlights/playerTracking/shot_42_tracking.json",
    "verticalURL": "sportsEngine/65cb1dfb0b7a241a9e448600/pixellot_ml/highlights/verticalView/shot_42_vertical.json"
  }
}
```
