---
title: 3D Slime Mold Simulation
date: 2021-03-31
description: Extending particle simulation to another a new dimension.
---

Slime mold simulation is an intersting topic which is best explained here: https://sagejenson.com/physarum. Essentially you simulate many particles moving in a 2D plane, the particles leave a trail of goop behind it and at each stage of the simulation the particles try to move in the direction of maximum goop. This causes the particles to flock together in interesting ways and trace organic shapes.

Last summer I wrote a simple physarum simulation using compute shaders in OpenGL. Some of the results were pretty interesting, and I experimented with having particles of different colors that tried to separate from each other. 

I started to try and use Niagara, the new vfx system in Unreal Engine, and given its capabilities I thought it would be interesting to recreate a physarum simulation and possibly extend it to work in the THIRD DIMENSION.

The setup to get things working in Niagara was a bit confusing since I had to use Simulation Stages, which is a powerful feature, but doesn't always work as expected.

This video helped me get started https://www.youtube.com/watch?v=XVKpofOj44c

In order to simulate slime mold we need particles that move around according to our rules, and we need a grid that we can write data into to store the density of the particle trails. 

There are two modes to the simulation stages. A stage's iteration source can be the particles or a data interface. If it is particles then the stage executes some code for each particle that currently exists. If it is a data interface and the interface is a grid then it executes some code for each cell in the grid. 


# Volumetric Physarum
To make our simulation volumetric we need to swap out our 2D texture for a 3D one. This way our particles will move around in 3D space and leave a trail inside of a volume. This involves doing a few new things. First, rendering a volume as not as simple as rendering a 2D texture. There are multiple ways to render a volume, but I wrote a shader to raymarch and estimate the density to use that as the alpha. 

The other thing we need to do is redesign how our particles moved. Previously they just sampled to locations to the left, right, and in front, but we want to make use of this extra dimension so we need to sample more points. The simplest method I could think of is just to sample left, right, forward, up, and down, but using a diffrent sampling method could yield more interesting results. In 2D we just use a single angle to determine which direction the particle was facing, but now we'll need method, either a quaternion or basis vectors. To keep things simple I just stored a forward vector and use that to calculate the up and right vectors when I needed to rotate the particles.



```c++
void light() {
    float brightness = dot(LIGHT, NORMAL);
    // DIFFUSE_LIGHTING is the return value of this function
    DIFFUSE_LIGHTING += vec3(brightness);
}
```

