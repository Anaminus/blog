+++
title = "Input handling"
date = 2023-06-28 21:39:05
tags = ["roblox", "gamedev"]
+++

Certain input types have to be handled in certain ways. Key repetitions must be
handled by monitoring the key's InputObject, while mouse wheel input is best
handled by getting it from a source, because an emission from a source doesn't
always correspond to a property change.

![](00.mp4)
