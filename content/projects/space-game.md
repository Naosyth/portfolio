---
title: "Space Game (CS 381)"
date: 2013-05-10T12:00:00-07:00
draft: false
---

[Space Game on GitHub](https://github.com/Naosyth/cs381game)

## About This Project

This is my final project for CS 381 (Game Engine Development). It is a three dimensional game set in space, where the player must protect a cargo ship that is under attack, as it flies towards a jump game. The gameplay is not very imaginative, as the main goal of this project was to implement the control math for character / camera movement and rotation. This was my first time working in depth with quaternions, which can seem intimidating at first, but are actually pretty nice to work with.

## Technology

This game was built using [Python-Ogre](http://wiki.ogre3d.org/Python-Ogre), which handeled the rendering pipeline. Everything else was built from scratch using Python. The models were exported from the Tribes 2 Construction Mod using a script I wrote several years before this project, which converted structures built in-game to `.obj` files. These were then cleaned up and given very basic textures with [Blender](https://www.blender.org/).

{{< figure src="https://i.imgur.com/WD0XIVS.png" link="https://i.imgur.com/WD0XIVS.png" width="700px" title="Jump Gate in Blender" caption="This is an example of a model that was built in the Tribes 2 Construction Mod and then exported to Blender.">}}

{{< figure src="https://i.imgur.com/E3KRqpm.png" link="https://i.imgur.com/E3KRqpm.png" width="700px" title="Player's Ship in Blender" caption="This model was also exported to Blender, but some features were added in Blender such as the thrusters.">}}

## Demo Video

{{< youtube id="2zB9NIcwB-k" >}}
