---
title: Raymarching in Unreal Engine 4
date: 2020-07-20
---

<iframe width="420" height="315" src="https://www.youtube.com/embed/dQw4w9WgXcQ" frameborder="0" allowfullscreen></iframe>

An Image:
![my vent](/assets/imgs/vent.jpg)
Raymarching is a fascinating tool, useful for rendering a wide variety of effects. However, it is not particularly easy to use raymarching in Unreal, since it does not provide easy access to a shader scripting language, instead you have to create shaders using the visual material editor. There is a node that allows you to enter custom HLSL code into though, which we will be taking advantage of.

I am not going to explain raymarching in-depth, so if you want to learn more this VIDEO explains it well, and this ARTICLE offers some easy to understand shader examples.

We will be placing our raymarching shader onto a cube, so that it can be placed and viewed in the scene. This method was adapted from this VIDEO, which uses method and implements it in Unity.

To begin we will start writing our raymarching functions in a custom shader node. To be able to write functions in the custom node requires a bit of a hack, which results in some strange looking code, if you want an explanation of why you write it this way look HERE at this ARTICLE.

We will write all our raymarching and signed-distance functions in global node that we can use later. First we will write the raymarch function which takes in the camera position and the view direction towards the current pixel and returns the distance to the current point from the camera.

To start writing HLSL functions add the following to your custom node:

```c
// this escapes from the current HLSL function and allows us to write our own functions
	return 1;
}

float raymarch( float3 camPos, float3 rayDir) {
    // raymarching code goes here

// note the lack of closing bracket
```



Now we can actually write the raymarcher:

```c
// Parameters: Camera Position and Ray Direction
// Output: Distance to closest surface along ray
float raymarch( float3 camPos, float3 rayDir ) {
    float depth = 0;

    for ( int i = 0; i < MAX_STEPS; i++ ) {
        float3 pos = camPos + ( rayDir * depth );
        dist = sceneSDF( pos );

        depth += dist;
        if ( dist <= EPSILON || depth > MAX_DIST ) {
            break;
        }
    }
    return depth;
}
```

You can define `MAX_DIST`, `MAX_STEPS`, and `EPSILON` with:

```c
#define MAX_DIST 100.0
#define MAX_STEPS 100
#define EPSILON 0.0001

float raymarch( float3 camPos, float3 rayDir ) {
 	...
    ...
```
