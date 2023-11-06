---
layout: post
title: "4D Games: Problems with Ray Marching"
description: "4D Games uses ray marching to render the 4D shapes. In this post I explain what I did to optimize my shaders, and solve some visual artefacts."
---

- [What is Ray Marching](#what-is-ray-marching)
- [4D Artefacts](#4d-artefacts)
  - [Valley Boosting](#valley-boosting)
- [Optimizing the 4D Maze](#optimizing-the-4d-maze)
  - [Using a Bounding Box](#using-a-bounding-box)
  - [Using a Minimum Required Step Distance](#using-a-minimum-required-step-distance)
  - [4D Ray Tracing](#4d-ray-tracing)

## What is Ray Marching

<p style="text-align: justify">
The shapes in 4D Games are rendered using ray marching. Ray marching is the process of stepping forward along a ray, where each step is the distance from the last point along the ray, to the nearest surface.
</p>

<img src="{{ '/assets/figures/algorithm_ai.png' | absolute_url }} " alt="ray-march-algorithm" style="max-width: 50%;margin-left: auto;margin-right: auto;"/>

```glsl
// MAX_DIST: Maximum distance from the ray origin
// SURF_DIST: distance from the surface before we consider ourselves "Touching" the surface
// MAX_STEPS: Maximum number of times we can step forward along the ray

// ro: Ray Origin
// rd: Ray Direction
float Raymarch(float4 ro, float4 rd){
    float dO = 0; // Distance from Origin
    float dS;     // Distance from Surface

    for(float i=0; i < MAX_STEPS; i++ ){
        float4 p = ro + rd * dO;
        dS = GetDistance(p);
        dO += dS;
        if (dS < SURF_DIST || dO > MAX_DIST) break;
    }
    return dO;
}
```

<p style="text-align: justify">
This algorithm has to be done for every pixel on the screen, which quickly becomes very computationally expensive. This is why we limit the number of steps the function can take. Unfortunately, reducing the number of steps the ray marcher can take can lead to distortion and artefacts.
</p>

## 4D Artefacts

<p style="text-align: justify">
Here is the Pentachoron (5-Cell). Using the algorithm above, with MAX_STEPS = 120, we get this artefact. If we move far enough along the W axis, to where the shape is no longer in our 3D slice, we can still see a dark shape something on screen.
</p>

<img src="{{ '/assets/gifs/Artefacted5Cell.gif' | absolute_url }} " alt="artefact.gif" style="max-width: 70%;margin-left: auto;margin-right: auto;"/>

<p style="text-align: justify">
The reason we see this artefact is because as the ray passes the surface, the distance function still returns a small distance. Because of this, the point hardly marches forward and quickly uses up whatever remaining steps it had.
</p>

<img src="{{ '/assets/figures/ArtefactProblem.png' | absolute_url }} " alt="ray-march-artefact" style="max-width: 60%;margin-left: auto;margin-right: auto;"/>
<p style="text-align: center"><i>
MAX_STEPS = 10. Although this ray should pass this surface, it is only able to take small steps, quickly using up the step count, and marking this pixel as hitting a surface.
</i></p>

### Valley Boosting

<p style="text-align: justify">
I haven't been able to find a good solutions to this problem, probably because this only becomes a big problem when viewing a cross section of a higher dimensional shape.
</p>

<p style="text-align: justify">
My solution to the problem is what I call "Valley Boosting". Valley Boosting works by comparing your current distance to the surface with your previous distance. If this distance is greater, you can be described as moving over a hill, into a valley. We assume that there is probably a greater distance to travel ahead, and don't want to constantly be checking what we have already passed. In this case, we give the march forward a bonus distance.
</p>

<img src="{{ '/assets/figures/ValleyBoost.png' | absolute_url }} " alt="ray-march-valley-boosting" style="max-width: 60%;margin-left: auto;margin-right: auto;"/>

<p style="text-align: center"><i>
MAX_STEPS = 10. Using Valley Boosting, the ray marcher is able to boost past the second half of the hill, shown by the two green circles and yellow bars. 
</i></p>

```glsl
// MAX_DIST: Maximum distance from the ray origin
// SURF_DIST: distance from the surface before we consider ourselves "Touching" the surface
// MAX_STEPS: Maximum number of times we can step forward along the ray
// BONUS_DIST: The bonus step size for Valley Boosting

// ro: Ray Origin
// rd: Ray Direction
float Raymarch(float4 ro, float4 rd){
    float dO = 0; // Distance from Origin
    float dS;     // Distance from Surface
    float prevDS = MAX_DIST; // Previous Distance from Surface

    for(float i=0; i < MAX_STEPS; i++ ){
        float4 p = ro + rd * dO;
        dS = GetDist(p);
        if (dS >= prevDS)
        {
            dS = max(dS, BOOST_DIST);
        }
        prevDS = dS;
        dO += dS;
        if (dS < SURF_DIST || dO > MAX_DIST) break;
    }
    return dO;
}
```

<p style="text-align: justify">
Now the 5-cell instantly vanishes when moved far enough along the W axis. No more artefacts!
</p>

<img src="{{ '/assets/gifs/ValleyBoosted5Cell.gif' | absolute_url }} " alt="valley-boost.gif" style="max-width: 70%;margin-left: auto;margin-right: auto;"/>

---

## Optimizing the 4D Maze

*Taking the maze from 5fps to 60fps*

<p style="text-align: justify">
The 4D maze is made up of 4D-capsule paths. The larger mazes can have > 3000 paths. Even after culling paths on different slices, you still have to ray march over hundreds of paths about 100 times for every pixel, every frame. The performance wasn't looking too good... About 5fps at 100% resolution for the \(8^{4}\) maze and 30fps at 40% resolution...
</p>

### Using a Bounding Box

<p style="text-align: justify">
The first step to improve performance was to minimize the evaluations of the actual paths as much as possible. The first bunch of steps are essentially wasted marching towards a cube. It is only when the ray has entered the bounding cube do we need to start evaluating all the paths of a maze.
</p>

<p style="text-align: justify">
The distance function of the maze was structured such that it would march towards a cube, and if inside of the cube, it would check the paths for the maze. This gave about a 10% improvement in performance.
</p>

```glsl
GetDistance(float4 p)
{
    const float distanceToMaze = sdBox(p-_MazeBoundingBoxOffset, _MazeBoundingBoxOffset) - playerRadius;
    if (distanceToMaze > SURF_DIST+0.1f) return distanceToMaze;

    ...Player sphere and maze paths...

    return d;
}
```

### Using a Minimum Required Step Distance

<p style="text-align: justify">
Next I wanted to try reduce the amount of steps necessary for a good looking maze render. One common way to do this is to force a minimum step size to prevent wasted steps making very small progress.
</p>

<p style="text-align: justify">
The way I chose to implement this for the maze was to only use the minimum step size unless the previous step has put us in a position where we will now be within the surface distance.
</p>

<p style="text-align: justify">
As the ray marcher was now making more progress through the maze faster, I could reduce the value of MAX_STEPS. This worked decently, giving a small boost in performance, but not much.
</p>

```glsl
// MAX_DIST: Maximum distance from the ray origin
// SURF_DIST: distance from the surface before we consider ourselves "Touching" the surface
// MAX_STEPS: Maximum number of times we can step forward along the ray
// MIN_DIST: Minimum distance every step

// ro: Ray Origin
// rd: Ray Direction
float Raymarch(float4 ro, float4 rd)
{
    float dO = 0.0; // Distance from Origin
    float dS;       // Distance from Surface

    for (float i = 0.0; i < MAX_STEPS; i++) {
        float4 p = ro + rd * dO;
        dS = GetDistance(p);
        if (dS < SURF_DIST)
        {
            dO += dS;
            break;
        }
        dS = max(dS, MIN_DIST);
        dO += dS;
        if (dS < SURF_DIST || dO > MAX_DIST) break;
    }
    return dO;
}
```

### 4D Ray Tracing

<p style="text-align: justify">
Ultimately I settled on Ray Tracing for the maze which ended up being much easier than I expected. Ray tracing requires you to be able to calculate the intersection point of a ray through a shape. Commonly this is done with spheres and triangles. This process is much more complicated than creating arbitrary distance functions for ray marching. Fortunately, intersection functions already exist for cylinders/capsules and spheres, so recreating the maze was fairly straightforward.
</p>

<p style="text-align: justify">
The maze using ray tracing now always runs above 60fps at 100% resolution for the \(8^{4}\) maze on my computer!
</p>

<p style="text-align: justify">
I know this solution didn't really have a satisfying pay off for how to optimize ray marching for thousands of shapes, but I hope at the very least my attempts can help if you find yourself working on something similar.
</p>

<!--
<center>
    <video width="100%" controls>
        <source src="/assets/devlog/maze_effect.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>
</center>
-->