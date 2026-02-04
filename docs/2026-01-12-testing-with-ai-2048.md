---
title: "My plan"
---

How do I prepare a language model to test a specific application? I need application data to feed into the model. I need to collect application event logs, plenty of them. To collect this data, I need full access to the source code specifically to add instrumentation.

The application to be tested should not be too simple but also not too complex in order to avoid spending too much time on application tracing and training. So I decided to go for an open source game: [2048](https://github.com/gabrielecirulli/2048). It's a game where you combine matching tiles to create a new tile with the sum of their values. If you reach 2048, you are the winner! 

Since the game is open source it is perfect for my experiment. After I have added event logging to code I can launch and run the game locally. Maybe I can also host the game to let my friends or family members play the game and help me in collecting usage data.

Here is the [2048 forked repo](https://github.com/palapiessa/2048)