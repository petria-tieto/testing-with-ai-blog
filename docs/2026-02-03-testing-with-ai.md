---
title: "Using the Model to Play"
---
In last blog I trained the model with logs from my own game sessions. In this blog we will implement a server and an autoplay script for playing the game with the model. Soon you will discover how does the model perform: did it beat my score?

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true });</script>

```mermaid
sequenceDiagram
    participant Browser as Browser UI
    participant Game as GameManager (JS)
    participant Autoplay as autoplay.js
    participant Server as Flask Model Server
    participant Model as Random Forest.pkl

    Browser->>Game: Render 2048 board
    Browser->>Autoplay: User calls autoplay.start()
    Autoplay->>Game: captureSimpleState()
    Game-->>Autoplay: { grid, score }
    Autoplay->>Server: POST /predict (grid, score)
    Server->>Model: load/predict_proba(features)
    Model-->>Server: move probabilities
    Server->>Autoplay: { move, valid_moves, ... }
    Autoplay->>Game: move(directionIndex)
    Game->>Browser: Update board + score
    Autoplay->>Autoplay: Schedule next poll
```
````