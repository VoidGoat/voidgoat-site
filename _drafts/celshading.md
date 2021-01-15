---
title: Toon Shading
date: 2021-01-20
description: Explore the possibilties of toon shading in 3D rendering.
---

{% assign tempPath = page.id | split: '/' | last %}
{% assign img = "assets/imgs/" | relative_url | append: tempPath  %}

Toon shading or cel shading can give 3D objects a striking stylized look. In order to implement toon shading we will be using a custom light shader. In order to create a toon shading model we need to understand how to create basic realistic shading model. To calculate lighting for any particular pixel we need two vectors. The normal vector which points perpendicular to the objects surface at the given pixel, and the light vector which points from the surface to light source. The surface should be most bright when the normal vector and light vector are pointing in the same direction, that is when the surface is facing the light. The surface should be darkest when the normal vector is pointing away from the light vector. To get a value describing whether the two vectors are pointing more in the same direction or more in opposite directions we can use the dot product.

The shaders I am writing are in the Godot Engine, but the same principles apply to any 3D Rendering engine. 


This gives us the simplest implementation of a traditional shading model:
```c++
void light() {
    float brightness = dot(LIGHT, NORMAL);

    // DIFFUSE_LIGHTING is the return value of this function
    DIFFUSE_LIGHTING += vec3(brightness);
}
```

To turn this into toon lighting we need to discretize the output so we get distinct areas bands of shadow and light rather than a smooth gradient.
