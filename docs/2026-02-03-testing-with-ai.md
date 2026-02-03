---
title: "Using the Model to Play"
---
In the last blog post I trained the model with logs from my own game sessions. In this blog we will implement a server and an autoplay script for playing the game with the model. Soon you'l see how the model performs: does it beat my score?

Below you will see a diagram of the autoplay architecture. It has 3 important parts:
- the game itself
- Python logic with Flask server
- autoplay.js module that drives the game

Autoplay.js runs in the browser, grabs the board state from the GameManager instance, and makes an HTTP POST to the Flask service (model_server.py). The server loads the random-forest pickle, returns the best move in JSON, and autoplay.js feeds that move back into gameManager.move(...) See the diagram below. 

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true });</script>

<div class="mermaid">
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
</div>

Here are the script paths in my 2048 source:
````markdown
service/
├── board_rules.py: logic for allowed moves
├── model_server.py: server and model driver
└── test_valid.py: lightweight test harness, not in use
test/
└── autoplay.js: autoplay communication included to index.html
````

To run the autoplay game mode, first start the Flask service:
- ```RF_ALLOWED_ORIGINS=http://127.0.0.1:5500 PORT=5050 python3 model_server.py``` 
- RF_ALLOWED_ORIGINS is given to avoid CORS related issues
- run the command in python venv, see setup instructions in [README.md](https://github.com/palapiessa/testing-with-ai-blog/blob/main/README.md) 
- the script assumes model file ```random_forest_2048.pkl``` in same directory

Open the game's ```Index.html``` in your favorite browser. 

Launch the autoplay mode from browser's developer tools:
- Safari: Develop - Show Javascript Console
- Below an image from Safari:

<img src="./images/2026-02-03-autoplay-start.png" alt="js console" width="400" />

How did the model perform? To be honest, the scores were not that good but it was expected since it was learning from my own logs. :)
But the most important thing is that the model ***can*** play the game! 
Here is a video of one game session:

<video controls width="420">
    <source src="./images/2026-02-03-autoplay-recording.mp4" type="video/quicktime" />
    Your browser does not support the video tag.
</video>

In today's blog post I have covered how the autoplay feature is implemented and used to run the game. This is great but what is the value of this feature? Stay tuned; in next blog I will cover how this feature can be used in testing the game.