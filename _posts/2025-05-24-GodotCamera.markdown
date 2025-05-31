---
layout: post
title:  "Recreating the Pokemon Snap Camera"
date:   2025-05-24 16:00:00 +0000
categories: Godot
---

I had a plan for a game a while ago in which you play as a photographer working for a newspaper company. Your goal is to wander around, interviewing people, investigating, and most importantly taking photos. While I don't have a full concept ready, the first step is to get the Camera working. 

<a href="https://github.com/JamieBali/pokemonSnapCamera"> Godot code is available here.</a>

# Research

My plan for how this system will work is based on the camera from Pokemon Snap. In Pokemon Snap, you travel around a safari course with the goal of taking pictures of various Pokemon. While Nintendo are very hush-hush about exactly how their system works, I can make an educated guess as to how the system works.

When the "take picture" button is pressed, a series of rays will be fired out from the player camera. Each of these rays will eventually colide with either a Pokemon or an environment object. We can then sum up the objects detected to determine what proportion of the image contains specific features. 

Pokemon Snap's camera system has a couple more features too, primarily that it is able to identify the pose of the Pokemon, the direction they're facing, and much more. For the time being, my system will not include these features, but I want to ensure that it's easy enough to modify this system to include these features in the future.


# Design

The plan for my system is a series of raycasts within the camera's field-of-view which will return a dictionary object containing every object colided with and the number of detections which were found. Each item in the environment will then have an attribute attached, such as "tree", "ground", "person", or whatever else I need to identify in images. 








<img src="https://imgur.com/vX0QSzf.png" height="200px">

{% highlight python %}
{ "Ground": 458, "sky": 728, "Cobblestones": 78, "Hedge": 133, "Bench": 56, "TrashCan": 21, "Streetlamp": 26 }
   At: res://characterHandlers/playerCameraHandler.gd:29:get_objects()
{% endhighlight %}