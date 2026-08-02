# **Basketball ML Breakdown \- Player Highlights Glossary**

This document describes the highlights detected in basketball ML breakdown. The system uses machine learning to automatically identify and timestamp key player actions during basketball games.

## **Player Highlights**

| Attribute | Type | Description | Values |
| :---- | :---- | :---- | :---- |
| eventId | string | Unique identifier for the basketball game/event |  |
| hlsUrl | string | HTTP Live Streaming URL for accessing the game video |  |
| sport | string | Sport type identifier | basketball |
| schemaVersion | string | Version of the schema specification | v1.0.0 |
| schemaUrl | string | Reference URL to the complete schema definition |  |
| processedAt | string | Timestamp when the ML analysis was completed |  |
| jerseyColor | string | Player's jersey color in hexadecimal format |  |
| jerseyNumber | number | Player's jersey number as displayed on uniform |  |
| startTime | number | Start time of the highlight in seconds from game beginning |  |
| endTime | number | End time of the highlight in seconds from game beginning |  |
| type | string | Category of the detected highlight | shot, assist |

## **Value Descriptions:**

**Type Values:**

* shot: A successful shot attempt by a player  
* assist: A pass that directly leads to a teammate's successful shot
