---
weight: 4
title: "Shortest path-finding with Dijkstra's algorithm"
date: 2019-12-01T21:57:40+08:00
draft: false
author: "MW"
authorLink: "https://MarkWentink.github.io/DSED"
description: "This article shows a basic implementation of Dijkstra's algorithm."
images: []
resources:
- name: featured-image
  src: maze.jpg
- name: featured-image-preview
  src: maze.jpg
tags: ["Python", "Algorithms"]

lightgallery: true
---

In my previous post, I described how **Dijkstra's algorithm** allows us to make our way through mazes, maps, and other graphs, finding *the shortest path between a start and end point.*

Let's have a look here at what an implementation of that could look like. 

<!--more-->

To illustrate, I will use a challenge from my favourite coding event, **Advent of Code**. 

### Advent of Code

Every year, [Advent of Code (AoC)](https://adventofcode.com/2022/day/12) publishes a coding challenge every day of December leading up to Christmas. The challenges can be solved in any language, and typically involve algorithmic or dynamic programming to solve within reasonable time. I find them tho be great ways to hone my coding skills!

Most years, Dijkstra's algorithm comes in handy on one day or another. Below is the example of AoC 2022, day 12.


> You are given a heightmap of the surrounding area (your puzzle input). The heightmap shows the local area from above broken into a grid; the elevation of each square of the grid is given by a single lowercase letter, where a is the lowest elevation, b is the next-lowest, and so on up to the highest elevation, z.
Also included on the heightmap are marks for your current position (S) and the exit (E). Your current position (S) has elevation a, and the exit (E) has elevation z.
You'd like to reach E, but to save energy, you should do it in as few steps as possible. During each step, you can move exactly one square up, down, left, or right. To avoid needing to get out your climbing gear, the elevation of the destination square can be at most one higher than the elevation of your current square; that is, if your current elevation is m, you could step to elevation n, but not to elevation o. (This also means that the elevation of the destination square can be much lower than the elevation of your current square.)


This is a perfect use case for Dijkstra's algorithm! We need to find the shortest path from S to E. 

To up the challenge a bit, we have some restrictions on how we are allowed to travel: We can take one step up, down, left or right at a time, and we can either stay on the same elevation, go up **one** level, or go down as many as we like.

Let's first make this a bit more concrete by visualising the output:


```python
import numpy as np

def load_data():
    '''
    The input for AoC challenges are always unique to each player, and are given as .txt files. 
    To easily work with the data later on, we will read it from the text file and store each character as the corresponding ASCII code in a numpy array cell.
    '''
    data = []
    with open('day12.txt') as f:
        for line in f.readlines():
            data.append([ord(x) for x in line.strip()])
    return np.array(data)
```


```python
data = load_data()

for i in range(len(data)):
    print("".join([chr(x) for x in data[i]])) # Convert back to letters for the visualisation
```

    abcccccaaaaaacccaaaccaaaaaaaacccaaaaaaccccccccccccccccccccccccccccaaaaaaaaaaaaaacacccccccccccccccccccccccccccccccaaaaaaaacccccccccccccccccccccccccccccccccccccccccccccaaaaa
    abcccccaaaaaaaacaaaaccaaaaaaccccaaaaaaccccccccccaaacccccccccccccccaaaaaaaaaaaaaaaacccccccccccccccccccccccccccccccaaaaaaaaaccccccaaaccccccccccccccccccccccccccccccccccaaaaaa
    abccccaaaaaaaaacaaaaccaaaaaaccccaaaaaaaaccccccccaaaccccccccccccccccaaaaaaaaaaaaaaccccaaaccccccccccccccccccccccccccaaaaaaaaccccacaaaccccccccccccccccaaccccccccccccccccaaaaaa
    abcccaaaaaaaaaacaaaccaaaaaaaacccccaaccaaacaaccccaaaaaaaccccccccccccaacaaaaaaaaaccccccaaaaccccccccccccccccccccaaacaaaaaaccccccaaaaaaaacccccccccccccaaaacccccccccccccccaaacaa
    abcccaaacaaaccccccccaaaaaaaaaaccccccccaaaaaacaaaaaaaaaaccccccccccccccaaaaaaaaaaccccccaaaaccccccccccccccccaaccaaacaaaaaaacccccaaaaaaaacccccccccccccaaaaccaaaccccccccccccccaa
    abcccccccaaaccccccccaaaaaaaaaaccccccaaaaaaaccaaaaaaaaacccccccccccccccaaaacaaaaaaaacccaaaaccccccccccccccccaaaaaaacaaccaaacccccccaaaaacccccccccccccccaaaaaaaaacccccccccccccaa
    abcccccccaacccccccccacaaaaaccaccccccaaaaaaacccaaaaaaaccccccaaacccccccaacacaaaaaaaacccccccccccccccccccccccaaaaaaccccccaaaccccccaaaaaccccccccccccjjkkaaaaaaaacccccccccccccccc
    abaccccccccccccccccccccaaaacccccccccccaaaaaaccccaaaaaaccccaaaacccccccccccccaaaaaaccccccccccaccaaccccccccccaaaaaaaaccccccccccccaaaaaaccccccccccjjjkkkkkaaaaccccccccaaccccccc
    abaccccccccaaaccccccccccaaccccccccccccaacaaacccaaaaaaaccccaaaacccccccccccccaaaaaaccccccccccaaaaacccccccccaaaaaaaaaccccccccccccaccaaacccccccccjjjjkkkkkkkaacccccccccaccccccc
    abaacccccccaaaaccccccccccaaaccccccccccaacccccccaaaacaaccccaaaacccccccccccccaaaaaacccccccccccaaaaaccccccccaaaaaaaacccccccaaccccccccccccccccccjjjjoooookkkkkllllllccccaaacccc
    abaacccccccaaaaccccccccccaaaaccccccccccccccccccaacaaacaaaccccccccccaaccccccaaacaaccccccccccaaaaaacccccccaaaaaaaccccccccaaaacccccccccccccccccjjjoooooopkkkklllllllccccaacccc
    abaccccccccaaacccccccccccaaaacccccccccccccccccccccaaaaaaacccccccccaaaccccccccccccccccccccccaaaaccccccaaaccccaaaccccccccaaaacccccccccccccccccjjooooooopppklppplllllcccaccccc
    abacccaaaaaccccccccccccccaaaccccccccccccccccccccccaaaaaaccccccaaaaaaaccccccccccccccccccccccccaaacccccaaacacccaaccccccccaaaaccccccccccccccccjjjooouuuupppppppppplllcccaccccc
    abccccaaaaacccccccccccccccccccccccccccccccccccccaaaaaaaaccccccaaaaaaaaaaccaacccccccccccccccccccccccaacaaaaaccccccccccccccccaaacccccaccaccccjjjoouuuuuupppppppppllllccaccccc
    abcccaaaaaacccccccccccccaaccccccaaacccccccccccccaaaaaaaaaccccccaaaaaaaaacaaaaccccccccccccccccccccccaaaaaaaaccccccccccccacccaaccccccaaaacccjjjjoouuuuuuupuuuvvpqqlllccaccccc
    abcccaaaaaaccccccccccccaaacaacccaaaaacccccccccccaaaaaaaaaaccccccaaaaaaaccaaaacccccccccccccccccccccccaaaaaccccccccccccccaaaaaaacccccaaaaaccijjooouuuxxxuuuuvvvqqqlmccccccccc
    abcccaaaaaaccccaacccccccaaaaaccaaaaaccccccccccccaaaaaacaaacccccaaaaaaccccaaaacccccccccccccccccccaaaccaaaaaccccaaaccccccaaaaaaaacccaaaaaaciiiinootuxxxxuuyyyvvqqqmmccccccccc
    abcccacaacccccaaacaaccaaaaaacccaaaaaccccccccccccaaaaaacccccccccaaaaaaaccccccccccccaaccacccccccccaaacaaacaaccaaaaaccccccaaaaaaaaaccaaaaaaiiinnnnnttxxxxxyyyyvvqqqmmdddcccccc
    abcccaaacaaacccaaaaaccaaaaaaaaccaaaaacccccccccccaaacaaaccccccccaaccaaaccccccccccccaaaaaccccccaaaaaaaaaacccccaaaaaacccccaaaaaaaaaccccaaciiinnnnttttxxxxxyyyyvvqqmmmdddcccccc
    abcccaaaaaaacaaaaaacccaacaaaaaccaacccccccccccaaaaaaaaaaaccccccccccccaacccccccccccaaaaacccccccaaaaaaaacccccccaaaaaacccccaaaaaaaaccccccciiinnnnttttxxxxxyyyyvvqqqmmmdddcccccc
    SbcccaaaaaaccaaaaaaaaccccaacccccccccccaacccccaaaaaaaaaaccccccccccccccccccccccccccaaaaaaccccccccaaaaaccccccccaaaaacccccaaaaaaaccccccccciiinnntttxxxEzzzzyyvvvqqqmmmdddcccccc
    abccaaaaaaaacaacaaaaacccaaccccccccccccaaacccccaaaaaaaaaaaccccccccccccccccccccccccccaaaacccccccaaaaaaccccccccaaaaaccccccacaaaaccccccccciiinnntttxxxxxyyyyyyvvvqqmmmdddcccccc
    abcaaaaaaaaaacccaacccccaacccccccccacacaaaccccccaaaaaaaaaacccccccccccccccccccccacccaaccccccccccaaaaaaccccccccccccccccccccaaaaaccccccccciiinnntttxxxxyyyyyyyyvvvqqmmmdddccccc
    abcaaaaaaaaaaccaaccaaaaaacccccccccaaaaaaaaaacccaaaaaaaaaacaaccccccccccccaaaaaaaaccccccccccccccaccaaaccccccccccccccccaaacaaacccccccccaaiiinnnttttxxwwyyyyyyyvvvqqmmmdddccccc
    abaaccaaacaaaccccccaaaaaaaccccccccaaaaaaaaaacccaaaaaaaacccaaaccccccccccccaaaaaacccccccccccccccccccccccccccccccccccccaaaaaaaaaacccaaaachhhnnnntttsswwyywwwwwvvvrrqkmdddccccc
    abaaacaaacccccccccccaaaaaaacccccccccaaaaaacccccaacccaaacccaaaaaaaccccccccaaaaaaccccccccccccccaaccccccccccccccccccccccaaaaaaaaacccaaaaaahhhmmmmmsssswwywwwwwwvrrrrkkdddccccc
    abaaaaaaacccccccccccaaaaaaacccccccccaaaaaaccccccaaccccccaaaaaaaaccccccccaaaaaaaaccccccccaaccaaaccccccccccccccccaacccccaaaaaaacccccaaaaahhhhhmmmmssswwwwwrrrrrrrrkkkeeeccccc
    abcaaaaaaccccccccccaaaaaacccccccccccaaaaaacccccaaaaccccaaaaaaaaacccccccaaaaaaaaaacccccccaaacaaacccccccccccccacaaaccccaaaaaaccccccaaaaacchhhhhmmmmsswwwwrrrrrrrrrkkkeeeccccc
    abaaaaaacccccccccccaaaaaaccccccccccaaaaaaaaccccaaaaccccaaaaaaaaccccccccaaaaaaaaaacaaacccaaaaaaccccccaacccccaaaaaccaccaaaaaaacccccaccaacccchhhhmmmssswwsrrrrrrrkkkkkeeeccccc
    abaaaaaaaccccccccaaccccaaccccccccccaaaaaaacccccaaaacccccccaaaaaacccaaccacaaaaaccccaaaccccaaaaaaaaaacaaaccccaaaaaaaaccaaacaaaccccccccccccccchhhhmmssssssrlllkkkkkkkeeecccccc
    abaaaaaaaaccccccaaacaacccccccccccccaaaaaaaaccccccccccccccaaaaaaacccaacccccaaaaccccaaaaaaaaaaaaaaaaaaaacccccccaaaaaccccccccaaaacccccccccccccchhgmmmsssssllllkkkkkeeeeeaacccc
    abcaaacaaacccccccaaaaaccccccccccccaaaaaaaaaccccccccccccccaaaccaaaaaaaaaacccaaccaaaaaaaaaaaaaaaaacaaaaaaaccccaaaaacccccccaaaaaacccccccccccccccggmmmlssslllllffeeeeeeeaaacccc
    abcaaacccccccccaaaaaacccccccaaacaaacaaaaaaacccccccccccccaaaaccccaaaaaaaacccccccaaaaaaaaaaaaaaaccaaaaaaaaccccaacaacccccccaaaaaacccccccccccccccgggmmlllllllfffffeeeeaaaaacccc
    abcaaccccccccccaaaaaaaacccccaaaaaaaaaaaaaccccccccccccccccaaaacccccaaaacccccaacccaaaaaaaccccaaaccaaaaaaaacccccccaaccccccccaaaaaaaccccccccccccccggglllllllffffffecacaaacccccc
    abcccccccccaaccaacaaaaaccccccaaaaaaaaaaaaccccccaaccaaccaaaaaaccccaaaaaccccaaccccccaaaaaacccaaaccaacaaaccaaaacccccccccccccaaaaaaaccccccccccccccggggllllfffffcccccccaaacccccc
    abcccccccaaaaaaccaaacccccccccaaaaaaaaccaaacccccaaaaaaccaaaaacccccaaaaaacccaaaccaaaaaaaaacccccccccccaaaccaaaaacccccccccccaaaaaaaaaccaaccccccccccggggggfffffccccccccccccccccc
    abcaaccccaaaaaaccaaccccccccaaaaaaaaaacccaacccccaaaaaccccaaaaaccccacccaaccaaaaccaaaaaccaacccccccccccccccaaaaaacccccccaaacaaaaaaaaaaaaacccccccccccgggggfffaacccccccccccccaccc
    abaaaccccaaaaaaccccccccaaacaaaaaaaaaaaaaaaaaaccaaaaaacccaaccacccccccaaaaaaaaaaaaaaaccccccccccccccaaaaccaaaaaacccccccaaaaaaaaaacaaaaaaccccccccccccagggfcaaaccccccccccccaaaaa
    abaaaacccaaaaaccccccccaaaacaaacaaacccaaaaaaaacaaaaaaaaccccccccccccccaaaaaaaaaaaaaaacccccccccccccaaaaacccaaaaaccccccccaaaaaacaaaaaaaaccccccccccccccaccccaaacccccccccccccaaaa
    abaaaaccccaaaaccccccccaaaacccccaaacccccaaaacccaaaaaaaacccccccccccccccaaaaaaaaaaaaaacccccccccccccaaaaaaccaaaccccccccccaaaaaaaaaaaaaaaaccccccccccccccccccccacccccccccccccaaaa
    abaacccccccccccccccccccaaacccccaacccccaaaaaacccccaacccccccccccccccccccccaaaaaaaaacccccccccccccccaaaaaacccccccccccccaaaaaaaaaaaaaaaaaaacccccccccccccccccccccccccccccccaaaaaa

`a` here corresponds to the lowest elevation, on which level our starting point `S` is. Our end point `E` is on the highest elevation, `z`.

In graph theory speak, we refer to points or locations as *nodes*, which is the term I will adopt here. We will refer to these nodes as *visited* when we have processed them.

### Process

We will take the following steps in our implementation:

1. Make a list of all the nodes we haven't visited yet. At the start, this is all of them. 
2. Create a dictionary of nodes to record their corresponding shortest distance from the start. These will all start at infinity, and get updated whenever we find a shorter path to that node.
3. Set the distance dictionary entry for the starting point to 0.
4. Set the starting point as the 'current' node.
5. In a loop, while the end node hasn't been visited yet:
    - Find unvisited neighbours to the current node
    - For each one, check whether they are accessible (i.e. their elevation is at most 1 higher than the current node)
    - If accessible, we can take a step there, meaning their distance from the starting point (via this path) is one further than the current node. If that distance is smaller than what is curently recorded in the dictionary, overwrite it.
    - Remove the current node from the 'not visited' list
    - Choose the next current node: Out of those that we haven't visited yet, pick the one with the smallest recorded distance.  
6. By the end of the loop, the dictionary entry for the end point will be the nr of steps required to get there via the shortest path. 

We'll first convert the elevation to numbers 1-26 for easier comparison, and then we will unleash Dijkstra's algorithm to calculate the distance from start to exit. The ascii code for `a` is `97` and the rest of the lowercase alphabet is consecutive after that, so we will subtract 96 from every entry to turn it into a `1-26` range.


```python
data = data - 96
```

1. Make a list of all the nodes we haven't visited yet. At the start, this is all of them.

Our map is rectangular, so we can generate a list of tuple coordinates by borrowing the `product()` function from `itertools`


```python
import itertools

not_visited = list(itertools.product(np.arange(data.shape[0]),np.arange(data.shape[1])))
```


```python
# Spot check some entries
print(not_visited[:10])
print(not_visited[510:520])
```

    [(0, 0), (0, 1), (0, 2), (0, 3), (0, 4), (0, 5), (0, 6), (0, 7), (0, 8), (0, 9)]
    [(2, 168), (2, 169), (2, 170), (3, 0), (3, 1), (3, 2), (3, 3), (3, 4), (3, 5), (3, 6)]


That looks good. We start in the top left corner with coordinates `(0, 0)`, make our way to `(0, 170)`, and then wrap around to `(1, 0)` and so on.

 2. Create a dictionary of nodes to record their corresponding shortest distance from the start. These will all start at infinity, and get updated whenever we find a shorter path to that node.


```python
distances = {}
for node in not_visited:
    distances[node] = np.inf
```

3. Set the distance dictionary entry for the starting point to 0.

After setting our entries `a-z` to numbers `1-26`, our starting point `S` has now shifted to number `-13`


```python
start = np.where(data == -13)
start_loc = list(zip(start[0], start[1]))[0]
distances[start_loc] = 0

print(f"Starting point at {start_loc}. Distance set to {distances[start_loc]}")
```

    Starting point at (20, 0). Distance set to 0


Our starting point is halfway down the first column on our map. Presumably the exit will be somewhere on the right. We need to locate it before we can start our Dijkstra loop. As with the `S` for start, our `E` for end has shifted, and we are looking for number `-27`


```python
end = np.where(data == -27)
end_loc = list(zip(end[0], end[1]))[0]

print(f"End point at {end_loc}. Current distance is {distances[end_loc]}")
```

    End point at (20, 146). Current distance is inf


Currently, the distance to our end point is infinity, because we have no idea how to get there. Time to start searching!

Now that we have located the start and end points, we need to reset their elevations to play nice when compared to the other nodes, and then our final step will be to set the start as our current node.


```python
data[20, 0] = 0 # Reset starting location to lowest elevation
data[20, 146] = 26 # Reset end location to highest elevation
```

4. Set the starting point as the 'current' node.



```python
current = start_loc
```

5. In a loop, while the end node hasn't been visited yet:


```python

while end_loc in not_visited:

    # Find unvisited neighbours to the current node. To keep things simple, we will test the four possible neighbours separately.

    
    # Top neighbour: we need to check that current node isn't already in the top row, 
    # and that the top neighbour wasn't visited yet
    if (current[0]-1 >= 0) and (current[0]-1, current[1]) in not_visited:
        
        # Check that the elevation change is allowed 
        if data[current[0]-1, current[1]] - data[current] < 2:
            
            # If so, calculate the distance value for the neighbour 
            distance = distances[current] + 1
            
            # If that distance value is smaller than what was currently recorded, update the dictionary.
            if distance < distances[(current[0]-1, current[1])]:
                distances[(current[0]-1, current[1])] = distance
                
                
    # Bottom neighbour: we need to check that current node isn't already in the bottom row, 
    # and that the bottom neighbour wasn't visited yet
    if (current[0]+1 < data.shape[0]) and (current[0]+1, current[1]) in not_visited:
        if data[current[0]+1, current[1]] - data[current] < 2:
            distance = distances[current] + 1
            if distance < distances[(current[0]+1, current[1])]:
                distances[(current[0]+1, current[1])] = distance
                
                
    # Left neighbour: we need to check that current node isn't already in the leftmost column, 
    # and that the left neighbour wasn't visited yet
    if (current[1]-1 >= 0) and (current[0], current[1]-1) in not_visited:
        if data[current[0], current[1]-1] - data[current] < 2:
            distance = distances[current] + 1
            if distance < distances[(current[0], current[1]-1)]:
                distances[(current[0], current[1]-1)] = distance
                
                
    # Right neighbour: we need to check that current node isn't already in the righmost column, 
    # and that the right neighbour wasn't visited yet
    if (current[1]+1 < data.shape[1]) and (current[0], current[1]+1) in not_visited:
        if data[current[0], current[1]+1] - data[current] < 2:
            distance = distances[current] + 1
            if distance < distances[(current[0], current[1]+1)]:
                distances[(current[0], current[1]+1)] = distance
                


    # Remove current node from not_visited
    not_visited.remove(current)

    # Pick the next current node from not_visisted with the smallest distance
    # We need to account for the case where we may have visited every location and ran out of next options.
    if not_visited != []:
        current = min(not_visited, key=distances.get)
    

```


```python
print(f"The final distance for the end point is {distances[end_loc]} steps, which was the solution to this problem.")
```

The final distance for the end point is 520 steps, which was the solution to this problem.

