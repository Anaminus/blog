+++
title = "Input visualizer"
date = 2023-06-27 22:53:12
tags = ["roblox", "gamedev", "fusion"]
+++

Widget to visualize how InputObjects are produced. Whenever a new object is
made, it is added to the list, then monitored for changes. Each
Source+UserInputType+KeyCode combination produces its own object. Sources used
are the Input signals from UserInputService and a Frame GUI.

![](00.mp4)
