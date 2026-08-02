# **Outdoor Football ML Breakdown \- Team Highlights Glossary**

This document describes the highlights detected in outdoor football ML breakdown. The system uses machine learning to automatically identify and timestamp key team plays during outdoor football games.

## **Team Highlights**

| Attribute | Type | Description | Values |
| :---- | :---- | :---- | :---- |
| eventId | string | Unique identifier for the football game/event |  |
| hlsUrl | string | HTTP Live Streaming URL for accessing the game video |  |
| sport | string | Sport type identifier | Outdoor Football |
| schemaVersion | string | Version of the schema specification | v1.0.0 |
| schemaUrl | string | Reference URL to the complete schema definition |  |
| processedAt | string | Timestamp when the ML analysis was completed |  |
| start | object | Timestamp marking the beginning of the play |  |
| end | object | Timestamp marking the end of the play |  |
| is_scoring | boolean | Whether the play resulted in a score |  |
| is_explosive | boolean | Whether the play was an explosive gain |  |
| td | string or null | Touchdown indicator for the play |  |
| gain | number or null | Yards gained on the play |  |
| quarter | number or null | Quarter of the game the play occurred in |  |
| clock | string or null | Game clock time when the play occurred |  |
| down | number or null | Down number when the play occurred |  |
| yardLineBegin | number, string or null | Yard line where the play began |  |
| teamWithBall | string or null | Team in possession of the ball for the play | Home, Away, Unknown |
| playType | string or null | Category of the play | pass, run |
| result | string or null | Outcome of the play |  |
| distance | number or null | Distance needed for a first down |  |
| points_scored | number or null | Points scored on the play |  |
| is_turnover | boolean | Whether the play resulted in a turnover |  |
| turnover_type | string or null | Type of turnover, if any |  |
| total_rushing_yards | number | Team's total rushing yards for the game |  |
| total_passing_yards | number | Team's total passing yards for the game |  |
| final_score | number | Team's final score for the game |  |

## **Value Descriptions:**

**PlayType Values:**

* pass: A passing play  
* run: A rushing play

**TeamWithBall Values:**

* Home: The home team has possession of the ball  
* Away: The away team has possession of the ball  
* Unknown: Possession could not be determined
