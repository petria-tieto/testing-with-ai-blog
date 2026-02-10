---
title: "Game Over!"
---
We have a working AI player for 2048, but how does it perform? More importantly, can we trust the game engine itself? In this post, we're shifting from simply playing the game for fun to testing it seriously. We will use our AI to hunt for bugs.

Today I’m adding an end-to-end (E2E) testing harness that checks both move validity and the correctness of the game-over state. The harness lives in two helper files:
- `autoplay.js`
- `test_autoplay.py`

Autoplay.js contains a test-hook harness which activates only when specially required for  testing sessions and can be excluded from production builds. There are several existing  objects or functions modified at runtime by assigning new behavior to them. Methods like HTMLActuator.prototype.actuate, window.requestAnimationFrame, and window.fetch are wrapped or replaced so that, in addition to their original duties, they also capture game state for the test harness. This is called "monkey-patching" and is a quick way to instrument code without changing the application’s source files directly.

The tests use Playwright's Python bindings and Pytest. Pytest acts as the test runner and handles the setup fixtures. In this configuration, the game runs on a local static server with autoplay enabled, while Playwright controls the browser interactions.

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true });</script>

<div class="mermaid">
flowchart TD
    subgraph Playwright Test
        T[test_autoplay_moves_remain_valid]
        H[Init scripts & page listeners]
        C[Loop: await __predictEvents]
        V[Assertions on board & moves]
    end

    subgraph Browser Page
        subgraph Autoplay Hooks
            A[autoplay.js controller]
            F[Fetch wrapper stores window.__predictEvents]
            B[__snapshotBoard captures grid]
        end
        G[GameManager applies move]
    end

    M[(Model Server /predict)]

    T --> H
    H --> A
    A --> F
    F --> M
    M --> F
    F -->|payload recorded| C
    C --> B
    B --> V
    A -->|move index| G
    G --> B
    V -->|determines next iteration| C
</div>

This diagram below shows the full handshake: pytest sets up Playwright, autoplay.js fetches moves from the model server, GameManager applies them, and the test samples the recorded events plus board snapshots for its assertions.

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true });</script>

<div class="mermaid">
sequenceDiagram
    participant Py as Pytest Playwright Test
    participant JS as autoplay.js Hooks
    participant GM as GameManager & UI
    participant MS as Model Server (/predict)

    Py->>Py: Warm-up /predict
    Py->>JS: Inject init scripts & listeners
    Py->>GM: Start autoplay + first step

    loop For each turn
        JS->>MS: fetch(grid, score)
        MS-->>JS: {move, valid_moves, predicted_invalid}
        JS->>GM: gameManager.move(moveIndex)
        GM-->>JS: Updated grid state
        JS-->>Py: Append payload to window.__predictEvents
        Py->>JS: Read __predictEvents[eventIndex]
        Py->>GM: Query __snapshotBoard()
        GM-->>Py: Grid, score, gameOver
        Py->>GM: movesAvailable()
        alt Invalid move changed board
            Py->>Py: pytest.fail()
        end
        alt Game over reached
            Py->>Py: Break loop
        end
    end

    Py->>Py: Final assertions on snapshot & movesAvailable()
</div>

To reproduce the run inside VS Code, install the dependencies:

```bash
uv pip install playwright pytest
uv run playwright install chromium
```
In order to enable VSCode Test Explorer to detect the test, a ```settings.json``` was added to 2048 repo's ```.vscode```--folder. The Test Explorer runs headless; use this CLI for a headed session: 
```bash
uv run pytest --headed --tracing=retain-on-failure --screenshot=only-on-failure test/e2e/test_autoplay.py -k test_autoplay_moves_remain_valid
```
The suite finishes in about 30 seconds and passes — so I intentionally inject a bug to showcase a failure scenario. Luckily GitHub Copilot is smart enough to understand my good intentions and is not considering this a malicious hack and implements a bug given in a following prompt: 

```text
Let's add an artificial bug to the 2048 game to demo a test failure. 
The bug shall introduce a game over situation after 10 moves.
```

<img src="./images/2026-02-10-game-over.png" alt="js console" width="400" />

This "Game Over" scenario summarizes the experiment: the code is now guarded against regressions that would otherwise infuriate players. :)

What makes this unique is that the AI model drives the execution. We didn't write explicit test steps—saving the most laborious part of the process. Given the random nature of 2048, a traditional scripted test would be brittle and difficult to maintain compared to this dynamic approach.

Thanks for following this series on 2048 game testing! I welcome your comments and ideas sent in LinkeIn. What comes next? Let's see ...




