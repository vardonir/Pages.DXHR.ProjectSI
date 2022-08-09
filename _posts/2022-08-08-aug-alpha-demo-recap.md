---
layout: post
title: "August 2022 Alpha Demo Recap"
categories: demo
tags: [ue-demo, helipad, vtol-interior, first-floor, security-office, sarif-hr]
image: 2022augalpha/20220807233832.png
---

[6th of August 2022, I made a post on reddit](https://www.reddit.com/r/Deusex/comments/whs7ox/a_first_demo_in_my_project_to_recreate/) about a little project that I've been keeping quiet about for more than a year. I know this is the first post in this blog, so please see the about page for more info about the project itself.

Anyway, for the past month, I've been working on the VTOL interior and I've gotten started on the helipad itself. I had the idea of throwing that together, added a camera and looked up how UE5's sequence editor works, and made this.

<iframe width="560" height="315" src="https://www.youtube.com/embed/hReMq_UZFhc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The helipad set is very much a work in progress; I had to use some surfaces from the Megascans library for the helipad surface itself to make the demo look presentable, since the UVs and textures of the actual helipad are broken and need to be fixed manually.

The light used in the demo is directional, I rotated it a bit as the animation ran. Everything else is standard - I kept exposure to manual, there's a skylight, directional fog, etc etc. I only got into UE5 about a few weeks ago so there's a lot of room for improvement.

### some rendered stills

![alt](assets/img/2022augalpha/20220807233913.png)

![alt](assets/img/2022augalpha/20220807234127.png)

#### comparison shot 1

The in-game shots are from DXHR-DC, running in all max settings on 1080p. Normally, I just take a bajillion screenshots of what I'm working on and pile them in a folder, but I couldn't do that in this scene so I had to record it.

![alt](assets/img/2022augalpha/20220807234310.png)
![alt](assets/img/2022augalpha/20220807234300.png)

#### comparison shot 2
![alt](assets/img/2022augalpha/20220807234505.png)

without external lighting:
![alt](assets/img/2022augalpha/20220807234456.png)

using a tinted spotlight: 
![alt](assets/img/2022augalpha/20220808174728.png)

### blender
![alt](assets/img/2022augalpha/20220808221736.png)
![alt](assets/img/2022augalpha/20220808221523.png)

fun fact: if you run Ninja Ripper on the VTOL scene, you actually get to see Jensen's jacket just hanging there next to him. The camera doesn't show it since the entire scene focuses on the two of them, but it's there, textures and all.
![alt](assets/img/2022augalpha/20220808221553.png)

![alt](assets/img/2022augalpha/20220808221816.png)

another fun fact: the VTOL has no fourth wall. I guess they never really made it for that scene since the camera doesn't really pan to that direction.
![alt](assets/img/2022augalpha/20220808221841.png)

If there are any other non-pre-rendered scenes inside the VTOL, please let me know.

### helipad

It's still on the very early stages so I'm still only 10% into texturing it, but I do have some shots of the meshes I have so far. I still have a couple more set pieces to process and align, including the hangar right outside that you can't access and some background bits like the water tower.

![alt](assets/img/2022augalpha/20220808222003.png)
![alt](assets/img/2022augalpha/20220808222024.png)

fun fact: As far as I know, there are no scenes with Malik in the VTOL that are not pre-rendered. Now, they mention in the dev commentary that the scene where Jensen walks in the VTOL to talk to Sarif for the first mission was not supposed to be pre-rendered, but it turns out that the interior of the VTOL does come with textures, as well.

![alt](assets/img/2022augalpha/20220808222316.png)

The SI building facade looks a bit like a Borg cube with just the bare textures lmao

![alt](assets/img/2022augalpha/20220808222430.png)

### preview of other stuff in progress 

The meshes for the security office are mostly there. For texturing, some of the decals and the glass windows need a lot of manual work, and there's a massive set piece with broken textures (more on that later).
![alt](assets/img/2022augalpha/20220808222750.png)

I have all the meshes for the first floor, everything is ripped and processed (more details on the pipeline later), I just need to piece it together. It's actually pretty easy, just very tedious.
![alt](assets/img/2022augalpha/20220808222935.png)

### bonus: a little something that's been driving me nuts

For the demo, I actually wanted to have Sarif in the VTOL as the camera pans around. His character model was easy to get. Getting his textures just right made me slam my head against the wall for two days.

![alt](assets/img/2022augalpha/20220808223145.png)

The engravings in-game look a lot deeper compared to anything I can set-up on UE5 and Blender. I tried cranking up the blue channel of the normal map to some ridiculous numbers and still nothing. I played around with the mask channels, still nothing. My material set-up needs a lot of work.

But I'm not going to dwell on it right now because it's not the main focus of this part of the project, but I still want to get it to work goddamnit.

Also, yes, it's brown, according to the in-game textures. The design on the finger joints remind me of [another famous fictional character with a right arm prosthetic.](https://en.wikipedia.org/wiki/Edward_Elric)

