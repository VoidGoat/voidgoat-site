---
title: Raymarching in Unreal Engine 4
date: 2020-07-20
description: Learn how to implement raymarching in Unreal Engine using HLSL in a custom shader node
thumbnail: RaymarchUnreal/thumbnail.png
---
Raymarching is a fascinating rendering technique, and is useful for rendering a wide variety of effects. However, it is not particularly easy to implement raymarching in Unreal Engine, since it does not provide easy access to a shader scripting language, instead you have to create shaders using the visual material editor. There is a node that allows you to enter custom HLSL code into though, which we will be taking advantage of.

I am not going to explain raymarching in-depth, so if you want to learn more this VIDEO explains it well, and this ARTICLE offers some easy to understand shader examples.

We will be placing our raymarching shader onto a cube, so that it can be placed and viewed in the scene. This method was adapted from this VIDEO, which uses the same method and implements it in Unity.

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
      	// Calculate next position along view ray
        float3 pos = camPos + ( rayDir * depth );

        // Calcualte shortest distance to scene at position
        dist = sceneSDF( pos );

        depth += dist;
      	// finish when very close to a surface or when ray has travelled to far
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

Now we need a `sceneSDF` function, so that we can calculate the distance to the surface at any point. The simplest SDF is sphere, so we can use that to start

```c
float sceneSDF( float3 pos ) {
 	return length(pos) - 40.0;
}

float raymarch(...) {
  ...
```

Now we can finally render something, although it won't look too impressive yet. First we need to add another custom HLSL node to the material graph, so that we can actually run the calculations.

```c
float depth = raymarch( CameraPosition, RayDirection );

// if no surface was hit then return black
if (depth >= MAX_STEPS) {
  return float4( 0.0, 0.0, 0.0, 0.0 );
}

// otherwise return white
return float4( 1.0, 1.0, 1.0, 1.0 );
```

If you've done everything correctly so far you should see a white circle in the middle of any mesh that you apply the material to. The circle is actually a sphere, but it doesn't have any shading, so it doesn't look like much.

## Calculating Normals

To shade the sphere properly we need to get the normals of the sphere, we could do this mathematically, but a more robust method is to estimate them by sampling the SDF.

Add the following function to the global custom node

```c
float3 estimateNormals(vec3 pos) {
  return normalize( float3(
  	sceneSDF( pos.x - EPSILON, pos.yz ) - sceneSDF( pos.x + EPSILON, pos.yz ),
    sceneSDF( pos.x - EPSILON, pos.yz ) - sceneSDF( pos.x + EPSILON, pos.yz ),
    sceneSDF( pos.x - EPSILON, pos.yz ) - sceneSDF( pos.x + EPSILON, pos.yz )
  ));
}
```

Now we can use this function to estimate the normal of a surface at any point. To use this we can add it to our final color calculation.

```c
if (depth >= MAX_STEPS) {
  return float4( 0.0, 0.0, 0.0, 0.0 );
}

// visualize the estimated normals
return calculateNormals( CameraPosition + (RayDirection * depth));
```

Now the material should look more like a sphere, since it should be displaying the surface normals as colors. However, it still doesn't really look like a real object, since it has no lighting.

## Phong Shading

We are going to use Phong shading, since it is fairly simple to implement. Now that we have the surface normals we can use them to calculate how much light would hit each part of the surface. We are going to implement a simple directional light, but you could add point or spot lights with the same method.

First we need a direction vector for our light

```c
float3 lightDir = normalize( float3( -0.3, -1.0, -0.3 ));
```
