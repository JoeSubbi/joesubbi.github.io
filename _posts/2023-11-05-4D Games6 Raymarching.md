---
layout: post
title: "4D Games: Problems with Ray Marching"
description: "4D Games uses ray marching to render the 4D shapes. In this post I explain what I did to optimize my shaders, and solve some visual artefacts."
logo: 
  path: "title3.png"
  alt: "4d-games-logo"
---

- [What is Ray Marching](#what-is-ray-marching)
- [4D Artefacts](#4d-artefacts)
  - [Valley Boosting](#valley-boosting)
- [Optimizing the 4D Maze](#optimizing-the-4d-maze)
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

Improving the maze

the maze can have hundreds to thousands of paths at once. Iterating over so many meant very slow performance

### Using a Minimum Required Step Distance

started by implementing min step distances

before or after algorithm

boosted performance but not enough

### 4D Ray Tracing

Using ray tracing for the Maze

60fps 100% resolution

<p style="text-align: justify">

</p>
