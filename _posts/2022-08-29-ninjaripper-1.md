---
layout: post
title: "NinjaRipper: Its use and issues, part 1"
categories: research
tags: [research-notes, ripping, ninja-ripper, mostly-rambling, blender, python]
image: ninjaripper/20220830001212.png
---

## intro

I love NinjaRipper. It's the only thing that made this entire project possible in the first place. But it's not perfect. It has its issues.

If you make a rip as-is, keep everything as defaults, the output won't look as good. Everything will be askew. 

The highlighted mesh is supposed to be the bounding box of a standard room, 4 walls and a floor and a ceiling.

![alt](assets/img/ninjaripper/20220830000805.png)

Look at those angles. That ain't right.

This is what the devs have to say about it:

> One of the peculiarities of the rip, is the distortion of the mesh geometry. You need to restore the geometry. One of the options is to find the projection matrix in the ripper's log, if it is not there you should try to adjust FOV manually.

Luckily, DXHR is one of those games where the projection matrix (which is a 4x4 matrix) is available in the log. But if you make one rip and look for 4x4 matrices, you get... a fcking lot of matrices. I found about ~1100 matrices in one log file for one rip. I'm not going to test all those matrices, especially since I can't automate that process (or, I haven't found a way to do so).

I asked in the channel. They gave me a matrix from a log I provided. Hooray. But I'm not going to need more than one rip (further details on why below) and I'm not going feed them logs for every single rip I make, so I needed to find my own solution.

And I did. I originally intended to put up my python code here on finding those bloody matrices, but it's 10% semi-logical conclusions and 90% handwavy "eh, it produces stuff that works," so I'm going to skip it.

But, tldr, if you want to use NR, you need that matrix.

## ok now what

When you make a fresh rip, with the right matrix, the output looks a bit like this:

![alt](assets/img/ninjaripper/20220829225758.png)
![alt](assets/img/ninjaripper/20220829225821.png)

Uh, the devs say that it's "shaders", like light effects and such. They're a bitch to clean up manually and in complex scenes, there tends to be a lot of them.

I have a code that cleans those up, but it's also a tad handwavy, I haven't fully tested its reliability, so I'd rather not discuss it here.

## duplicates? 

Run the rip, import it to Blender, get the output - there's two of the same mesh for some reason. Why? I dunno. I'm guessing it's a DXHR-specific thing, because I tried ripping another game and it didn't have that same issue. But the issue exists for my particular case and I need to deal with it.

I've noticed that the two overlapping meshes would have the same number of vertices, but that's all they have in common. Sometimes one mesh would have more textures, sometimes one would have different UV maps, etc. etc. What I did in the beginning was to go through every single mesh, find the one that looked "more correct", and then delete the other one.

After doing that for two days, I noped out pretty quickly. I'm a programmer, which means that I'm a lazy ass.

Here comes Blender's Python API to the rescue - I wrote a script that would go through each mesh in a rip...

```
objects = [o for o in context.scene.objects if o.type == 'MESH']
```

...get the vertices of that mesh as a numpy array...

```
def get_all_vertices(mesh):
    verts = [vert.co for vert in mesh.vertices]
    plain_verts = []
    
    for vert in verts:
        coords = []
        for coord in list(vert.to_tuple()):
            coords.append(round(coord))
        plain_verts.append(np.asarray(coords))
    
    return np.asarray(plain_verts)
```

...then compare each set of vertices to another set of vertices...

```
def vert_equiv(vert1, vert2):
    if vert1.shape == vert2.shape:
        return np.allclose(np.sort(np.ravel(vert1)), np.sort(np.ravel(vert2)), atol=1)
    else: 
        return False
    # return np.array_equiv(np.sort(np.ravel(vert1)), np.sort(np.ravel(vert2)))
```

(Side note, I originally used `array_equiv`, but it proved to be ineffective when it comes to aligning meshes from two separate rips. More on that later in this post)

...to every single other mesh in the rip. 

```
def populate_vert_list(object_list):
    vlist = [0] * len(object_list)
    
    for i in tqdm(range(len(object_list))):
        vlist[i] = get_all_vertices(object_list[i].data)
    return vlist

n = len(object_list)
obj_num = 0
dupe_mark = [True] * n

print("Calculating vertices")
vert_list = populate_vert_list(object_list)

print("Marking duplicates")
# go through all the objects in the scene and mark the duplicates
for i in tqdm(range(n)):
	if dupe_mark[i]:
		for j in range(i+1, n):
			if vert_equiv(vert_list[i], vert_list[j]):
				dupe_mark[j] = False

uniques = []
dupes = []
	
print("Separating uniques")
for i in tqdm(range(n)):
	if dupe_mark[i]:
		uniques.append(object_list[i])
	else:
		dupes.append(object_list[i])

duplicate_directory = [[i] for i in uniques]

print("Populating unique object vert list")
unique_vert_list = populate_vert_list(uniques)

print("Populating duplicate object vert list")
dupes_vert_list = populate_vert_list(dupes)

print("Building duplicate directory")
for u in range(len(uniques)):
	for d in range(len(dupes)):
		if vert_equiv(unique_vert_list[u], dupes_vert_list[d]):
			duplicate_directory[u].extend([dupes[d]])

print(f'{len(uniques)} uniques found')
```

(Side note: Could I have done this using pandas? Sure. But I wrote this code long before I found out how to setup a conda environment with Blender and I didn't fee like changing the code anymore, so... yeah.)

From there, I have a list of lists, where each sub-list is basically the same mesh grouped together. 

Then I go through that list, grab the mesh in each sublist with more UVs, then I move that to its own collection. I call those ones the 'dominant' meshes... for some reason (it was 2am and I was in the middle of packing to move to another apartment, cut me some slack). 

Why the UVs? In my particular case, it seemed that if there was two overlapping meshes, the one with more UV maps had a higher chance of having the correct UV map. A bit handwavy, but it works.

And then I took the texture(s) in the other mesh, called the 'non-dominant' mesh, copied that datablock to the corresponding dominant mesh, and then shoved the non-dominant mesh to its own collection, just in case I'll need it again, but I generally don't.

I also went through all the textures in each mesh and removed anything that was 4k-sized, 1080p-sized, and 1x1 pixels. I know for a fact that DXHR does not have any textures of that size, so those textures must have been some NR-related artifact. From what's left, I calculated the hash of the image and compared it with all the other image hashes in the texture. 

Voila - clean node trees, more or less. 

![alt](assets/img/ninjaripper/20220830001012.png)

Takes a bit of manual labor to link the right image to the Principled BDSF node, tho. 



## Alignment

So, now I can make a rip, clean it, fix it (mostly), but there's still the problem from the first part - it only gets the scene partially. 

![alt](assets/img/ninjaripper/20220815182242.png)
![alt](assets/img/ninjaripper/18256-0.gif)

![alt](assets/img/ninjaripper/20220815182320.png)
![alt](assets/img/ninjaripper/18256-1.gif)

You can see here that if you make a rip, you'd only end up with parts of the scene. This makes perfect sense - the game is not going to render anything that's outside of the camera's field of vision. But that means that I need to make a bunch of rips of the same location with different camera angles in order to get the full environment.

And that's fine.

Perfectly fine.

![alt](assets/img/ninjaripper/20220829215406.png)

(and that's just for the SI HQ maps - the Detroit map will probably be much larger)

And when it's imported, the meshes are rotated out of whack.

Note the XYZ axes om the upper right corner for all of these

![alt](assets/img/ninjaripper/20220829230531.png)
![alt](assets/img/ninjaripper/20220829231532.png)


The fix: for a pair of rips, I'd pick a mesh that's common to both rips, parent the entire rip to that mesh, and then align that to the destination rip using Mesh Align Plus or the the Align Objects function from the Tools:Q Animation kit by Project-StudioQ. The latter is more automatic (handles rotation, location, and scale), but it doesn't work all the time. The former is requires more manual checking but it generally works.

Uh. Video on request. Or when I get around to it.

BUT. It turns out that different rips, even when it *looks* exactly the same, would not produce the exact same mesh. There would be a very small difference between one rip from one view and another rip from another view, even if the mesh in question is the same one. 

Note the vertex positions in these two meshes

![alt](assets/img/ninjaripper/20220829232932.png)
![alt](assets/img/ninjaripper/20220829232943.png)

But the difference is so minute that you'd only see it if you were (1) looking for it specifically, or (2) you are a computer and all you see are the raw numbers, i.e., you're numpy's `array_equiv` function. Hence the change in that bit of the code.

I should point out that the difference is far far far more noticable in outdoor scenes, but that's not relevant right now.

## The output

With all that done, this is the output of 10 combined rips in the intro scene:

![alt](assets/img/ninjaripper/18256-09.gif)

(Manual stuff I needed to do here included deleting anything animated, mostly the humans and the robot arm that moves around. Seems like I did forget one human mesh, but eh...)
