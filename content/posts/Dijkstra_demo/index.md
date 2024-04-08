---
weight: 4
title: "Climbing Mountains: an implementation of Dijkstra's algorithm"
date: 2023-05-30
draft: false
author: "M Wentink"
authorLink: ""
description: "This article presents a basic implementation of Dijkstra's algorithm."
images: []
resources:
- name: featured-image
  src: maze.jpg
- name: featured-image-preview
  src: maze.jpg
#categories: ["Python", "Algorithms"]

lightgallery: true
---


In [my previous post]({{< ref "Path_finding" >}}), I described how **Dijkstra's algorithm** allows us to make our way through mazes, maps, and other graphs, finding *the shortest path between a start and end point.*

To illustrate what a code implementation might look like, I will use a challenge from my favourite coding event, **Advent of Code**. 

<!--more-->

### Advent of Code

Every year, [Advent of Code (AoC)](https://adventofcode.com/2022/day/12) publishes a coding challenge every day of December leading up to Christmas. The challenges can be solved in any language, and typically involve algorithmic or dynamic programming to solve within reasonable time. I find them tho be great ways to hone my coding skills!

Most years, Dijkstra's algorithm comes in handy on one day or another. Below is the example of AoC 2022, day 12.

### The Challenge

> You are given a heightmap of the surrounding area (your puzzle input). The heightmap shows the local area from above broken into a grid; the elevation of each square of the grid is given by a single lowercase letter, where a is the lowest elevation, b is the next-lowest, and so on up to the highest elevation, z. Also included on the heightmap are marks for your current position (S) and the exit (E). Your current position (S) has elevation a, and the exit (E) has elevation z. 
>
> You'd like to reach E, but to save energy, you should do it in as few steps as possible. During each step, you can move exactly one square up, down, left, or right. 
>
> To avoid needing to get out your climbing gear, the elevation of the destination square can be at most one higher than the elevation of your current square; that is, if your current elevation is m, you could step to elevation n, but not to elevation o. (This also means that the elevation of the destination square can be much lower than the elevation of your current square.)



This is a perfect use case for Dijkstra's algorithm! We need to find the shortest path from S to E. 

Note that we have some further restrictions on how we are allowed to travel: We can take one step up, down, left or right at a time, and we can either stay on the same elevation, go up **one** level, or go down as many as we like.

Let's first make this a bit more concrete by visualising the input. To make it easier to work with later, we will load in the data as numbers (matching the ASCII code for lowercase letters), but we will print them on screen as letters. Let's first write ourselves a function that loads the input data from a .txt file into a Numpy array.


```python
import numpy as np

def load_data():
    '''
    The input for AoC challenges are always unique to each player, and are given as .txt files. 
    To easily work with the data later on, we will read it from the text file and store each 
    character as the corresponding ASCII code in a numpy array cell.
    '''
    data = []
    
    with open('day12.txt') as f:
        for line in f.readlines():
            data.append([ord(x) for x in line.strip()])
            
    return np.array(data)

```


Next, let's check out the map we are given:


```python
data = load_data()

for i in range(len(data)):
    print("".join([chr(x) for x in data[i]])) # Convert back to letters for the visualisation
```

{{< image src="./map.png" height="400px" width="780px" >}}


That's a big mess of letters! I've highlighted the starting point `S` (which has elevation 'a'), and the end point `E` (elevation 'z') for clarity. Each letter corresponds to a position on this map. In graph theory speak, we refer to points or locations as *nodes*, which is the term I will adopt here. We will refer to these nodes as *visited* once we have processed them.

Now how do we get from Start to End across this mountain range of letters?

Let's consider the shortest path possible, just straight ahead from the start point:

{{< image src="./map_wrong.png" height="450" width="780" >}}

All is well for the first few steps. The first step requires moving from elevation `a` to `b`, which is allowed, and then from `b` to `c` next. We then stay on `c` for two steps and drop back down to `a`. Remember we are allowed to drop as many levels as we like. 

But after a few more steps we get stuck, as we would need to climb from `a` to `c` next, which is two steps, i.e. not allowed. 

So much for our straight path approach...

Ultimately, we are going to need to follow a path that winds around steep cliffs, slowly making its way across the mountain range. The red path below could be a good starting point, but ultimately, we need to find *the shortest path*.

{{< image src="./map_right.png" height="450" width="780" >}}

Let's harness Dijkstra's algorithm to make it happen!

Dijkstra's algorithm helps us calculate the shortest distance from every point on the map to the starting point. (We need to do an exhaustive search to ensure we have in fact the *shortest* path.)

A single step will be considered a distance of one. So all the neighbours to the starting point will be at distance 1, and a point diagonal from the starting point will be distance 2 (as we can only move up, down, left, right).

The exact steps are outlined below. 


### Implementation

**Note**: This section will read a lot easier if you browse [my introduction post on Dijkstra's algorithm]({{< ref "Path_finding" >}}), which goes into more detail of the algorithm itself. 

We will take the following steps in our implementation:

1. Make a list of all the nodes we haven't visited yet. At the start, this is *all* of them. 
2. Create a matching dictionary of nodes to record their corresponding shortest distance from the start. We will start these off as infinite distance, and update them whenever we find a shorter path from that node.
3. Set the distance dictionary entry for the starting point to 0.
4. Set the starting point as the 'current' node.
5. In a loop, while the end node hasn't been visited yet:
    - Find unvisited neighbours to the current node
    - For each neighbour, check whether they are accessible (i.e. their elevation is at most 1 higher than the current node)
    - If accessible, we can take a step there, meaning their distance from the starting point (via the current node) is one further than the current node. If that distance is smaller than what is curently recorded in the dictionary, overwrite it.
    - Remove the current node from the 'not visited' list.
    - Choose the next current node: Out of those that we haven't visited yet, pick the one with the smallest recorded distance.  
6. By the end of the loop, the dictionary entry for the end point will be the nr of steps required to get there via the shortest path. 

Before we start, we'll first convert the elevations to numbers 1-26 for easier comparison, and then we will unleash Dijkstra's algorithm to calculate the distance from start to exit. Currently, we have our letters stored as ascii codes. The ascii code for `a` is `97` and the rest of the lowercase alphabet is consecutive after that, so we will subtract 96 from every entry to turn it into a `1-26` range. This does mess up our `S` and `E` nodes, which we will reset later. 


```python
data = data - 96
```

------------

**1. Make a list of all the nodes we haven't visited yet. At the start, this is all of them.**

Our map is rectangular, so we can generate a list of tuple coordinates by borrowing the `product()` function from `itertools`


```python
import itertools

print(f"Map size: {data.shape}")

not_visited = list(itertools.product(np.arange(data.shape[0]),np.arange(data.shape[1])))

print(f"Top left: {not_visited[0]}")
print(f"Bottom right: {not_visited[-1]}")
print(f"Total nodes: {len(not_visited)}")
```

    Map size: (41, 171)
    Top left: (0, 0)
    Bottom right: (40, 170)
    Total nodes: 7011


That looks good. Our map is 41 nodes high and 171 nodes wide. We start in the top left corner with coordinates `(0, 0)`, make our way to `(0, 170)` in the top right, and then wrap around to `(1, 0)` and so on, all the way to `(40, 170)` in the bottom right.

------------

 **2. Create a dictionary of nodes to record their corresponding shortest distance from the start. These will all start at infinity, and get updated whenever we find a shorter path to that node.**


```python
distances = {}
for node in not_visited:
    distances[node] = np.inf

# If you are feeling adventurous, you can do this with a dictionary comprehension: 
# distances = {node:np.inf for node in not_visited}
```

We now have a dictionary that for each node (each location defined by a tuple of coordinates), records its distance from the starting point. At the start, we set all of them to infinite distance, as we don't know anything about possible paths yet. 

------------

**3. Change the recorded distance for the starting point to 0.**

We now need to find the coordinates of our starting point `S`, and set its distance to `0`.

After converting our entries `a-z` to numbers `1-26`, our starting point `S` has now shifted to number `-13`.


```python
# Find S
start = np.where(data == -13) 

# np.where returns a tuple of arrays, so we need to extract the coordinates first
start_loc = list(zip(start[0], start[1]))[0] 

# Edit the distance dictionary to set the starting point distance to 0.
distances[start_loc] = 0

print(f"Starting point at {start_loc}. Distance set to {distances[start_loc]}")
```

    Starting point at (20, 0). Distance set to 0


Our starting point is halfway down the first column on our map. The exit is somewhere on the right. We need to locate it before we can start our Dijkstra loop. As with the `S` for start, our `E` for end has shifted, and we are looking for number `-27`.

Once we have located the start and end points, we need to reset their elevations to play nice when compared to the other nodes, and then our final step will be to set the start as our current node.


```python
# Find E
end = np.where(data == -27)

# Extract coordinates
end_loc = list(zip(end[0], end[1]))[0]

print(f"End point at {end_loc}. Current distance is {distances[end_loc]}")

data[20, 0] = 0 # Reset starting location to lowest elevation
data[20, 146] = 26 # Reset end location to highest elevation
```

    End point at (20, 146). Current distance is inf


Currently, the distance to our end point is infinity, because we have no idea how to get there. Time to start searching!

------------

**4. Set the starting point as the 'current' node.**

To kick off the search loop, we will set the starting point as our current active node, and we'll start inspecting neighbours from there. 

```python
current = start_loc
```

------------

**5. In a loop, while the end node hasn't been visited yet:**

We identify allowed neighbours (i.e. not off the edge of the map, not already visited, and not too much higher in elevation), and for each check if a path via the current node, with distance current_distance + 1, is better than what was already recorded. Considering all nodes start at infinite distance, the first path towards them starts as the best path. Future paths will only overwrite this is their distance is shorter. 

Once we have processed all the neighbours, the current node is checked off the `not_visited` list, and a next current node is picked, from the `not_visited` list, based on which one currently has the lowest recorded distance. 

We keep going until we have processed the target node, `E` (`end_loc`).

```python
while end_loc in not_visited:


    neighbours = [(current[0]+i, current[1]+j) for (i,j) in [(-1,0), (1,0), (0, -1), (0, 1)]  # Generate all direct neighbours...
        if (current[0]+i in range(0, 41)  # as long as their row nr doesn't fall off the map...
        and (current[1]+j in range(0, 171))) # and their column nr doesn't fall off the map...
        and (current[0]+i, current[1]+j) in not_visited] # and the neighbour hasn't been visited yet.


    for neighbour in neighbours:
        
        # Check that the elevation change is allowed 
        if data[neighbour] - data[current] < 2:
            
            # If that distance value is smaller than what was currently recorded, update the dictionary.
            if distances[current] + 1 < distances[neighbour]:
                distances[neighbour] = distances[current] + 1

                
    # Remove current node from not_visited (i.e. tick it off the to-do list)
    not_visited.remove(current)

        # Pick the next current node from not_visisted with the smallest distance
        # We need to account for the case where we may have visited every location and ran out of next options.
    if not_visited != []:
        current = min(not_visited, key=distances.get)
        
```

Once the end node has been processed, we are confident that the value recorded for its distance is now the shortest one.


```python
print(f"The final distance for the end point is {distances[end_loc]} steps.")
```
    The final distance for the end point is 520 steps.

Note this doesn't tell us **what** the shortest path was, only how long it was. 

Although, to help visualise what we've ended up with, I've drawn out the (rather windy) path on our map. It looks like most of the path length comes from having to slowly spiral up the mountain on the right.

{{< image src="./map_complete.png" height="400px" width="780px" >}}

### You have arrived at your destination

Hopefully, you've seen that the implementation of Dijkstra's algorithm itself is not too laborious, but the trick is in formulating your problem in the context of nodes connected by some distance or cost. In the case of steps through a maze, it is easy: nodes are always 1 distance away from their neighbours. But think about a railway network of interconnected stations: every station will be some variable distance from its 'neighbours'. In this case, we would also need some lookup table of 'cost' to travel from one node to the next. 

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/)