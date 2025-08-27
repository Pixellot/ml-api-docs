```json title=Pixellot Stats API Schema"
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Pixellot Stats API Schema",
    "description": "Complete schema for the Pixellot Data API response including team stats, roster stats, box score, and game breakdown",
    "version": "1.0.0",
    "type": "object",
    "properties": {
      "schemaUrl": {
        "type": "string",
        "description": "URL reference to the current schema definition",
        "format": "uri",
        "examples": ["https://api.pixellot.com/schemas/stats/v1.0.0/schema.json"]
      },
      "schemaVersion": {
        "type": "string",
        "description": "Version of the schema following semantic versioning (X.Y.Z)",
        "pattern": "^\\d+\\.\\d+\\.\\d+$",
        "examples": ["1.0.0"]
      },     
      "matchTeamStats": {
        "type": "object",
        "description": "Stats for the teams that participated in the game",
        "required": [
          "metrics",
          "results"
        ],
        "properties": {
          "metrics": {
            "type": "object",
            "description": "Definitions of metrics used in the stats",
            "patternProperties": {
              "^[0-9]+$": {
                "type": "object",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "Full name of the metric",
                    "examples": ["2FG% By Team", "Points By Team"]
                  },
                  "shortName": {
                    "type": "string",
                    "description": "Abbreviated name of the metric",
                    "examples": ["2FG%", "P"]
                  },
                  "format": {
                    "type": "string",
                    "description": "Format to be used when displaying the metric value",
                    "examples": ["TWO_DECIMAL_POINTS", "TIME:HH:MM:SS"]
                  }
                }
              }
            }
          },
          "results": {
            "type": "object",
            "description": "The part of the response that contains the actual results, along with data explaining the context by which they were calculated",
            "additionalProperties": false,
            "required": [
              "entityHierarchy",
              "allStats",
              "includes"
            ],
            "properties": {
              "statsType": {
                "type": "string",
                "description": "Are the stats of type `allEntities` or `leaders`?",
                "enum": [ "StatsForLeaders", "StatsForAllEntities"]
              },
              "entityHierarchy": {
                "type": "array",
                "description": "The hierarchy used to group stats (e.g., [season, league, team]). The last entity is either `team` or `athlete`",
                "items": {
                  "type": "string",
                  "enum": ["season", "league", "game", "team", "athlete"]
                }
              },
              "allStats": {
                "type": "array",
                "description": "Array of stats groups",
                "items": {
                  "type": "object",
                  "description": "Represents a result of stats for a group of entities—e.g., stats for a team in a specific league in a specific season",
                  "properties": {
                    "parentEntities": {
                      "type": "object",
                      "description": "The IDs of the entities from `entityHierarchy`, minus the last item (the actual entity). Only the fields that appeared in `entityHierarchy` are present",
                      "properties": {
                        "season": {
                          "type": "integer",
                          "minimum": 1990,
                          "maximum": 2100,
                          "examples": [2020]
                        },
                        "league": {
                          "type": "string",
                          "examples": ["652d1a1d7720c86e48296b0d"]
                        },
                        "game": {
                          "type": "string",
                          "examples": ["652d1ad5d8733f6962e828ed"]
                        },
                        "team": {
                          "type": "string",
                          "examples": ["652d266bc4732ea5a44327f9"]
                        }
                      }
                    },
                    "stats": {
                      "type": "object",
                      "description": "The stats data structure varies based on statsType",
                      "if": {
                        "properties": {
                          "statsType": {
                            "const": "StatsForAllEntities"
                          }
                        }
                      },
                      "then": {
                        "patternProperties": {
                          "^[0-9a-fA-F]{24}$": {
                            "type": "object",
                            "description": "The key is the ID of the entity (team or athlete). The value is a map of metricId -> numeric result",
                            "patternProperties": {
                              "^[0-9]+$": {
                                "type": "number",
                                "format": "float"
                              }
                            }
                          }
                        }
                      },
                      "else": {
                        "if": {
                          "properties": {
                            "statsType": {
                              "const": "StatsForLeaders"
                            }
                          }
                        },
                        "then": {
                          "patternProperties": {
                            "^[0-9]+$": {
                              "type": "object",
                              "description": "The key is the ID of the metric for which we are calculating its leaders",
                              "patternProperties": {
                                "^[0-9a-fA-F]{24}$": {
                                  "type": "number",
                                  "format": "float",
                                  "description": "The key is the entity ID. The value is the numeric result for that metric"
                                }
                              }
                            }
                          }
                        }
                      }
                    }
                  }
                }
              },
              "includes": {
                "type": "object",
                "description": "The teams and/or athletes included in the aggregated stats",
                "patternProperties": {
                  "^[0-9a-fA-F]{24}$": {
                    "description": "The key is the ID of the entity (team or athlete); the value describes that entity",
                    "anyOf": [
                      {
                        "type": "object",
                        "properties": {
                          "objectType": {
                            "type": "string",
                            "description": "Type of the object in includes (in this case team)",
                            "const": "team"
                          },
                          "teamName": {
                            "type": "string",
                            "description": "The name of the basketball team",
                            "examples": ["Albert Einstein Girls Varsity Basketball"]
                          }
                        }
                      },
                      {
                        "type": "object",
                        "properties": {
                          "objectType": {
                            "type": "string",
                            "description": "Type of the object in includes (in this case athlete)",
                            "const": "athlete"
                          },
                          "firstName": {
                            "oneOf": [
                              {
                                "type": "string",
                                "description": "Player first name",
                                "examples": ["John"]
                              },
                              {
                                "type": "null"
                              }
                            ]
                          },
                          "lastName": {
                            "oneOf": [
                              {
                                "type": "string",
                                "description": "Player last name",
                                "examples": ["Doan"]
                              },
                              {
                                "type": "null"
                              }
                            ]
                          },
                          "middleName": {
                            "oneOf": [
                              {
                                "type": "string",
                                "description": "Player middle name",
                                "examples": ["Albert"]
                              },
                              {
                                "type": "null"
                              }
                            ]
                          },
                          "teamId": {
                            "oneOf": [
                              {
                                "type": "string",
                                "description": "Player team id",
                                "examples": ["652d266bc4732ea5a44327f9"]
                              },
                              {
                                "type": "null"
                              }
                            ]
                          },
                          "jersey": {
                            "type": "string",
                            "description": "Player jersey number",
                            "examples": ["23"]
                          }
                        }
                      }
                    ]
                  }
                }
              }
            }
          }
        }
      },
      "matchRosterStats": {
        "type": "object",
        "description": "Stats for the athletes on the roster that participated in the game",
        "required": [
          "metrics",
          "results"
        ],
        "properties": {
          "metrics": {
            "$ref": "#/properties/matchTeamStats/properties/metrics"
          },
          "results": {
            "$ref": "#/properties/matchTeamStats/properties/results"
          }
        }
      },
      "matchBoxScore": {
        "type": "object",
        "description": "Stats on the top players by different metrics",
        "required": [
          "metrics",
          "results"
        ],
        "properties": {
          "metrics": {
            "$ref": "#/properties/matchTeamStats/properties/metrics"
          },
          "results": {
            "$ref": "#/properties/matchTeamStats/properties/results"
          }
        }
      },
      "matchRosterLeaders": {
        "type": "object",
        "description": "Stats on the score of the game",
        "required": [
          "metrics",
          "results"
        ],
        "properties": {
          "metrics": {
            "$ref": "#/properties/matchTeamStats/properties/metrics"
          },
          "results": {
            "$ref": "#/properties/matchTeamStats/properties/results"
          }
        }
      },
      "gameBreakdown": {
        "type": "object",
        "description": "The raw data of the game, from which the stats are aggregated",
        "properties": {
          "results": {
            "type": "object",
            "properties": {
              "allStats": {
                "type": "array",
                "description": "Array of tag events that occurred during the game",
                "items": {
                  "type": "object",
                  "description": "A point of interest in a game which was logged, plus enriched data about the tag resource and tag attributes",
                  "required": [
                    "id", 
                    "gameId", 
                    "resource", 
                    "attrs",
                    "angles"
                  ],
                  "properties": {
                    "id": {
                      "type": "string",
                      "description": "The object id of the tag event",
                      "examples": ["67bbc79e91a779e0122dcf65"]
                    },
                    "gameId": {
                      "type": "string",
                      "description": "The object id of the game to which the event applies",
                      "examples": ["67bb4ef005aae699a0d9454e"]
                    },
                    "playclipId": {
                      "type": "string",
                      "description": "The object id of the play clip",
                      "examples": ["67bbc5b399fd464a56c9a6e6"]
                    },
                    "startUTC": {
                      "type": "integer",
                      "description": "The start of the tag event in unix UTC (only for Pixellot camera source or Replay kit)",
                      "examples": [1739552865]
                    },
                    "endUTC": {
                      "type": "integer",
                      "description": "The end of the tag event in unix UTC (only for Pixellot camera source or Replay kit)",
                      "examples": [1739552878]
                    },
                    "resource": {
                      "type": "object",
                      "description": "Details of a tag resource (used by a tag event)",
                      "additionalProperties": false,
                      "required": ["id", "name"],
                      "properties": {
                        "id": {
                          "type": "integer",
                          "description": "ID of the tag resource",
                          "examples": [157, 1000000]
                        },
                        "name": {
                          "type": "string",
                          "description": "Name of the tag resource",
                          "examples": ["Rebound", "Generic tag resource for playbased sports"]
                        }
                      }
                    },
                    "attrs": {
                      "type": "array",
                      "description": "Array of attributes that provide details about the tag event",
                      "items": {
                        "type": "object",
                        "description": "Details of a tag attribute (used by a tag event)",
                        "additionalProperties": false,
                        "required": ["id", "name", "type", "value"],
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "ID of the tag attribute",
                            "examples": [909, 779, 1444, 1446]
                          },
                          "name": {
                            "type": "string",
                            "description": "Name of the tag attribute",
                            "examples": ["Type", "Team", "Inning", "Pitcher"]
                          },
                          "type": {
                            "type": "string",
                            "description": "The type of the tag attribute value",
                            "enum": [
                              "text",
                              "number",
                              "list",
                              "teamNames",
                              "rosterAthletes",
                              "chartPoint",
                              "chartLine",
                              "rosterCoaches"
                            ]
                          },
                          "value": {
                            "description": "The value given to the attribute. For chartPoint, it will be a JSON string. For teamNames, it's a team ID. For rosterAthletes, it can be either an athlete ID or a string with jersey number and name",
                            "oneOf": [
                              { "type": "string" },
                              { "type": "number" },
                              { 
                                "type": "array",
                                "items": {
                                  "type": "string"
                                }
                              }
                            ],
                            "examples": [
                              "offense", 
                              "6579de1d7e14a67a0e1a91bf", 
                              "67a38884b62bf88b1146a82a", 
                              "50 John Cohen",
                              "{\"x\":517.2781982421875,\"y\":311.42251586914062,\"x2\":0,\"y2\":0,\"type\":\"point\",\"sector\":11,\"orientation\":\"left\"}"
                            ]
                          }
                        }
                      }
                    },
                    "angles": {
                      "type": "object",
                      "description": "Video offsets for different camera angles. The keys correspond to angle names, which are defined in the top-level angles array",
                      "additionalProperties": {
                        "type": "object",
                        "properties": {
                          "startOffset": {
                            "type": "number",
                            "description": "The start offset in seconds from the beginning of the video source"
                          },
                          "endOffset": {
                            "type": "number",
                            "description": "The end offset in seconds from the beginning of the video source"
                          }
                        },
                        "required": ["startOffset", "endOffset"]
                      },
                      "examples": [
                        {
                          "hd": {
                            "startOffset": 616,
                            "endOffset": 626
                          }
                        },
                        {
                          "HP": {
                            "startOffset": 82,
                            "endOffset": 95
                          },
                          "P3": {
                            "startOffset": 82,
                            "endOffset": 95
                          }
                        }
                      ]
                    }
                  }
                }
              }
            }
          },
          "includes": {
            "type": "object",
            "description": "Information about teams and athletes referenced in the tag events",
            "patternProperties": {
              "^[0-9a-fA-F]{24}$": {
                "oneOf": [
                  {
                    "type": "object",
                    "description": "Team information",
                    "properties": {
                      "teamName": {
                        "type": "string",
                        "description": "Name of the team",
                        "examples": ["Magdalena", "Creighton"]
                      }
                    },
                    "additionalProperties": false
                  },
                  {
                    "type": "object",
                    "description": "Athlete information",
                    "properties": {
                      "firstName": {
                        "type": ["string", "null"],
                        "description": "First name of the athlete",
                        "examples": ["Tate", "Wade"]
                      },
                      "lastName": {
                        "type": ["string", "null"],
                        "description": "Last name of the athlete",
                        "examples": ["Gillen", "Walton"]
                      },
                      "middleName": {
                        "type": ["string", "null"],
                        "description": "Middle name of the athlete"
                      },
                      "jersey": {
                        "type": "string",
                        "description": "Jersey number of the athlete",
                        "examples": ["20", "35"]
                      }
                    },
                    "additionalProperties": false
                  }
                ]
              }
            }
          }
        }
      },
      "angles": {
        "type": "array",
        "description": "All video angles/sources included in this game",
        "items": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string",
              "description": "Name of the camera angle",
              "examples": ["HP", "P3", "B1", "HH", "CF", "hd"]
            },
            "recordedStreamUrl" : {
              "type": "string",
              "description": "URL to the recorded video stream",
              "format": "uri",
              "examples": ["https://d2s0txx1aqnj4f.cloudfront.net/someTenantName/6736d0dc4c7136f9b31f0269/cloud_hls/0_hd_hls.m3u8"]            
            }
          }
        }
      }
    }
}
```
