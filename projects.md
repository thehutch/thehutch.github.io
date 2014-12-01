---
layout: page
title: Projects
permalink: /projects/
---

This section will contain projects I have made and what I am currently working on.

### Fusion Engine
[Github Source](https://github.com/thehutch/Fusion)

Fusion Engine is my custom Java game engine using OpenGL and GLSL.

### Atom Engine
Atom Engine is my custom cross-platform C++ game engine. Atom uses OpenGL3.3+ for rendering and the GLFW library to handle the window and input.

### A.T.L.A.S
ATLAS (Automatic Tag Learning Allocation System) is a project I developed for the BBC Technology Challenge. ATLAS uses Twitter's [HBC library](https://github.com/twitter/hbc) to track a specific phrase (Usually a hashtag) and count the number of times it is used. In addition to tracking a specific term, ATLAS can also search each tweet for more hashtags and count how many times they occur.

![ATLAS Console Window](/images/console.png)

I created a simple Javax Swing window to display this information which includes a console to print each incoming tweet along with its timestamp, and also a list of labels on the side which displays the hashtags which have been detects over a threshold amount.

If a hashtag is found more than a specified threshold then it is added to the right hand side of "trending" hashtags. In the example above the threshold was set to 15 to show that it worked. This would of course be set much higher in production use.

Once a message has been processed it is appended to the end of a file for storage. The data stored is only the timestamp of the tweet and the message, this is because there is no need to store any personal information.
Once ATLAS has finished processing, another appication could use the timestamps to display the rate of the tweets and can be useful for mapping at which points people were talking about the topic the most.

### University
Whilst at University I have developed some small games. These include a space car racing game and a 3D remake of the 1970's centipede with added features.
