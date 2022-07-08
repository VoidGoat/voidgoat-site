---
title: 3D Traveling Salesman Art
date: 2022-07-07
description: Explore how to create 3D TSP art using Blender and Python.
---

{% assign tempPath = page.id | split: '/' | last | append: '/' %}
{% assign img = "assets/imgs/" | relative_url | append: tempPath  %}



I recently had the idea of making traveling salesperson problem art in 3D. Typically, TSP art is made by taking a 2D image, distributing points on the image so that there are more points where the image is darker, and then trying to solve the traveling salesperson problem on that set of points. This creates some very interesting “connect-the-dots” style artwork. I thought that instead of using 2D points based on an image, it would be interesting to use points in 3D space based on a 3D mesh.

| ![Hands TSP ART]({{img}}hands-tsp.png) | 
|:--:| 
| *An example of TSP Art by Robert Bosch based on the Creation of Adam by Michelangelo. More examples can be found [here](https://www2.oberlin.edu/math/faculty/bosch/tspart-page.html).* |

## What It Is

The traveling salesperson problem is a mathematical optimization problem where you have a set of locations that a traveling salesperson needs to visit and you want to find the shortest route that includes all locations and returns the salesperson to their starting point.

| ![TSP visual explanation]({{img}}tsp-route-explainer.png){: .table-img } | 
|:--:| 
| *Randomly visiting points results in a very long route, but we can use a variety of algorithms to find more efficient routes.* |

## How I Started

To start implementing the idea, I used Blender so that I could easily create and modify 3D meshes and visualize the result of my algorithm in visually pleasing ways. Blender allows for Python scripting, so that is how I chose to implement my algorithm.

One of the troubles with creating TSP art is that the traveling salesperson problem is NP-Hard, meaning that, as far as humanity is aware, the only way to find the optimal route for the salesman is to check every possible option. However, the number of options grows at the rate of $$!(n-1)$$, so when you have 6 points to visit there are 120 options, not too bad, but when there are just 32 points to visit there are 8,222,838,654,177,922,817,725,562,880,000,000 options, making it essentially impossible to check them all and find the best solution. I want my algorithm to work on meshes with at least hundreds of points and ideally run in a few seconds rather than quadrillions of years, so the solution is to use heuristic algorithms that find solutions that are *pretty* good, but not perfectly optimal. This will work fine since we are trying to generate some artwork, so the most important thing is the circuitous look of the path, rather than whether it is actually the most efficient. So the question is, which heuristic algorithm to use?

## Genetic Algorithm, Shmenetic Shmalgorithm

Initially, after doing very little research, I was inspired by [this post](https://crescointl.com/using-a-genetic-algorithm-for-traveling-salesman-problem-in-python/) to use a genetic algorithm to solve the TSP. I adapted their code to 3D, (fixed some of the bugs) and then used the output to generate a network of edges based on a 3D mesh from Blender. The results were somewhat encouraging, but also disappointing. Running the algorithm on a simple, low-poly sphere yielded some interesting results, but the algorithm ran quite slowly and for more complex meshes did not find good solutions in a reasonable amount of time. It was exciting to see that solving the TSP in 3D yielded some interesting results, but I needed a better approach. Time to do some actual research.


| <video autoplay loop muted playsinline><source src="{{img}}genetic_tsp_result.mp4" type="video/mp4"></video> |
|:--:| 
| *The result of my experiments using a genetic algorithm. This is clearly not a great result, since you can see the path crossing over itself.* | 

I don’t think using a genetic algorithm is necessarily a bad approach but it requires a lot of fine tuning of parameters to get a good result in a reasonable amount of time and I wanted to investigate some other methods.


## Better Art or Better Algorithm?

After looking at some more advanced methods for solving TSP, I began to wonder whether that was the right approach at all. My goal was to make an interesting looking artwork and not to actually create a perfectly optimized path. This lead me to pursue algorithms that would produce a good look, rather than a better solution to the mathematical problem.

My first idea was to use the 2-Opt algorithm, which is one of the simplest ways to solve TSP. The 2-Opt algorithm starts with a random path and then iteratively looks at two points and the next points in the path and checks if swapping their end points would result in a shorter overall path. If the new path is shorter, then the edges are swapped. So if we have a path called $$p$$, then the algorithm looks at points $$p[x]$$ and $$p[y]$$ where $$p[x] → p[x+1]$$ and $$p[y] → p[y+1]$$, then the algorithm checks if the connections $$p[x] → p[y]$$ and $$p[x+1] → p[y+1]$$ would result in a shorter path. This process is repeated over and over until no improvement can be made. This is not actually a very effective solver for TSP since it almost always gets stuck in a local minima that it can never get out of. But it does produce a path that “looks” relatively nice. In 2D, the 2-Opt algorithm guarantees that no edges will cross, which results in one continuous line that you can easily follow with your eye. In 3D, it’s a little more complicated, but it yields a similarly neat result.

| <video autoplay loop muted playsinline><source src="{{img}}twoopt-tsp-result.mp4" type="video/mp4"></video> | 
|:--:| 
| *The result of using the 2-Opt algorithm on an icosphere. This is a much better result than the genetic algorithm and you can see there are no obviously inefficient parts of the path.* |

My initial implementation of 2-Opt was quite slow for larger meshes. My first run on a mesh with 500 vertices took an hour and forty minutes to run, so some optimizations were definitely necessary.

### Memoization Optimization

One of the best optimizations was caching all the distances between the points in a 2D array. The distances between points are constantly being compared, so not having to recalculate the distance saves a lot of time. Since the distance between points is always the same, they can just be looked up instead of being recalculated all the time.

Getting to a local minima solution where no improvements could be made for an Icosphere with 162 vertices took 2m19s without memoization and took about 16s with memoization. Definitely a big improvement.

### Route Comparison Optimization

There was another optimization that I initially ignored, but would make it much faster. I was previously calculating the length of the whole route to see if swapping the edges was an improvement. But in theory you only need to compare the edges being swapped and not the whole path which takes our distance calculation from $$O(n)$$ to $$O(1)$$ (explained here: [https://en.wikipedia.org/wiki/2-opt#Efficient_Implementation](https://en.wikipedia.org/wiki/2-opt#Efficient_Implementation)).  I couldn’t get this to work originally, because I was being stupid and was iterating over the points in their original order rather than in the order of the current route. But after implementing it, the Icosphere test dropped down from 16s to 4s. While memoization was just taking a constant time operation and making it faster, this change takes a linear time operation and makes it constant time, so for larger routes it should have an even bigger impact.

I am only trying to use this algorithm to generate a mesh that will be rendered later, so it does not need to run in real-time and these improvements are good enough.

## Making it Pretty

Since I implemented this algorithm in Blender, there are a lot of possibilities for how to render and visualize the data. I used Geometry Nodes to give the path some thickness and smoothed it out a little bit. I then experimented with different ways to show the path using Geometry Nodes and Shader Nodes.

## Results

After my Python script generates the basic mesh, I can use Blender’s Geometry Nodes to solidify it and give it some interesting animations. Then the result can be rendered out. Here is a video of some of my final results:

<iframe style="display: block; margin: auto;" width="560" height="315" src="https://www.youtube.com/embed/kvfetSDl8io" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Here are some still renders of some objects:

| ![Teapot TSP art]({{img}}teapot-tsp-still.png){: .table-img } | 
|:--:| 
| *It took about 18 min to calculate the route for this teapot with 821 vertices.* |

| ![Human TSP art]({{img}}human-tsp-still.png){: .table-img } | 
|:--:| 
| *An efficient route around the complexities of humanity.* |

| ![Suzanne Monkey TSP art]({{img}}suzanne-tsp-still.png){: .table-img }  | 
|:--:| 
| *You can't use Blender without using Suzanne in your results.* |


| ![2D Grid TSP Art]({{img}}2d-tsp.png){: .table-img } | 
|:--:| 
| *You can also still get some interesting results in 2D.* |

Overall I am quite happy with how it turned out and I think there are a lot of interesting areas to explore. If you make any 3D TSP art, I would love to see it. The code for this project and an example blend file are available on GitHub here: [https://github.com/VoidGoat/blender-3d-tsp-article](https://github.com/VoidGoat/blender-3d-tsp-article).

If you ever find yourself as a traveling salesperson underwater or in outer space, then perhaps these 3D algorithms and visualizations will help.

## Recommended Reading:

- [Opt Art by Robert Bosch](https://press.princeton.edu/books/hardcover/9780691164069/opt-art). Bosch started the idea of TSP art and his book goes into many other methods of creating art using optimization algorithms.
- [More 2D TSP Art by Robert Bosch](https://www2.oberlin.edu/math/faculty/bosch/tspart-page.html). Here you can see more traditional TSP art.
- [https://en.wikipedia.org/wiki/2-opt](https://en.wikipedia.org/wiki/2-opt)


---

You can subscribe to my newsletter to be notified when I release new articles and projects:

<a class="mail-widget" href="{{ '/newsletter' | relative_url }}">Get my newsletter<span
                class="fa fa-bubble fa-envelope"></span></a>

Or follow me on twitter:

<a href="https://twitter.com/VoidGoatDev?ref_src=twsrc%5Etfw" class="twitter-follow-button" data-size="large" data-show-count="false">Follow @VoidGoatDev</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
