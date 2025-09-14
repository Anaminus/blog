+++
title = "Random graphs"
date = 2023-01-12 17:24:25
tags = ["programming"]
+++

Random graph generation. Blue vertices have unexplored edges, while green ones
are completely explored. Occasionally connects a new vertex to a nearby existing
vertex, forming a loop. Rarely creates a long loop by connecting to the most
distant vertex.

{{< video src="00.mp4" >}}

The graph is represented by a force-directed graph that moves the vertices
around to make them easier to visualize. The actual graph is dimensionless, with
the vertices having no 2D or 3D position.
