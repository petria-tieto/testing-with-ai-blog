---
title: "Play for the data"
---
I consider myself a novice PC game player but luckily 2048 was not too hard to learn! :smile: I could reach up to number 512, it was hard to multiply that number. But luckily I didn't need to win the game in order to collect data for the model.

<img src="./images/2026-01-16-game-over.png" alt="My best score" width="400" />

It was quite easy to implement logging for every move, capturing the board state and current score. After asking an AI assistant for modeling ideas I decided to start with a RandomForestClassifier trained on the logs. It keeps things simple for this experiment; another option would be to explore a move-by-move LLM, maybe a future detour. In the next post I plan to share the script that actually trains the RandomForestClassifier on this data.

The logging logic is found in ```logger.js``` and the event logging calls hooked into ```game_manager.js```.

The schema for logging events is seen below. The most important item for training is the move-event because every move becomes a labeled row for the classifier.

```json
{
  "type": "array",
  "items": {
    "oneOf": [
      {
        "description": "The primary event used for model training",
        "type": "object",
        "properties": {
          "type": { "const": "move" },
          "ts": { "type": "integer", "description": "Timestamp" },
          "turn": { "type": "integer", "description": "Turn number" },
          "direction": { "type": "string", "enum": ["UP", "DOWN", "LEFT", "RIGHT"] },
          "valid": { "type": "boolean", "description": "Was the move legal?" },
          "prev": {
            "type": "object",
            "properties": {
              "grid": { "type": "array", "description": "4x4 board BEFORE move", "items": { "type": "array" } },
              "score": { "type": "integer" }
            }
          },
          "next": {
            "type": "object",
            "properties": {
              "grid": { "type": "array", "description": "4x4 board AFTER move", "items": { "type": "array" } },
              "score": { "type": "integer" }
            }
          }
        }
      },
      {
        "description": "Animation/UI event (ignored by training)",
        "type": "object",
        "properties": {
          "type": { "const": "moveTile" },
          "from": { "type": "object", "properties": { "x": {"type": "integer"}, "y": {"type": "integer"} } },
          "to": { "type": "object", "properties": { "x": {"type": "integer"}, "y": {"type": "integer"} } }
        }
      }
    ]
  }
}
```
Here is why this schema is helpful for the upcoming RandomForestClassifier training script:

- `prev` captures the full board and score before the move, which lets me engineer features like tile counts, merge opportunities, and positional heuristics.
- `direction` and `valid` give the target label and flags for filtering out illegal actions.
- `next` contains the post-move board and score so I can derive the score delta and outcomes such as new tile spawns for feature engineering.
- Non-move events are still logged but easy to filter out when building the training set.



Here is the grid state in JSON related to my best score game:
```json
{
  "ts": 1768479171162,
  "type": "gameEnd",
  "turn": 447,
  "reason": "loss",
  "score": 6476,
  "highestTile": 512,
  "state": {
    "grid": [
      [
        2,
        4,
        16,
        2
      ],
      [
        16,
        8,
        32,
        4
      ],
      [
        8,
        32,
        256,
        16
      ],
      [
        512,
        8,
        64,
        4
      ]
    ],
    "score": 6476
  }
}
```

There are strategies to follow to reach the biggest number, one of which is to keep big numbers towards bottom-right corner. As you see not very well adapted in this result.


From here the plan is to turn these logged moves into a feature matrix, fit the RandomForestClassifier, and compare its suggestions against my own play. Stay tuned for those results in the follow-up post.



