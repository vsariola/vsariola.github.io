Title: Fast Marching Methods on C64
Date: 2019-05-26
Category: programming
Summary: Implementation of Fast Marching Methods for the C64. One big trick to make it run in O(N) and a lots of small tricks to make the constant very small.

In 2007, while still a student, I learned of [Fast Marching Methods](https://en.wikipedia.org/wiki/Fast_marching_method), originally proposed by [J.A. Sethian](https://math.berkeley.edu/~sethian/). The algorithm operates on a grid, consisting of cells, each with different "slowness" (finite or infinite). The algorithm finds the shortest arrival time to each cell, similar to Djikstra's algorithm, but with a proper Euclidian norm. Applications of Fast Marching Methods include computing the shortest path for a robot navigating a maze or computing the shadows of point-like light sources. Which is to say, they are very useful in robotics or computer games.  

The running time of the original algorithm is O(N log N), where N is the number of cells visited. log N comes from the fact that the considered cells are kept in a priority queue.

Back then I learned that Yatziv, Bartesaghi and Sapiro had proposed a version that runs in O(N). Here the idea was that the priority queue was replaced with an "untidy priority queue": the considered cells are kept in a finite number of bins, but within each bin, there's no internal ordering. Instead of removing the smallest element from the queue, we remove any element from the smallest bin. The bins can be implemented as simple linked lists so adding and removing are O(1) operations and thus the total running time of the algorithm is O(N).

As a part of my studies, I was to take a special assignment in Computational Science and Engineering. I proposed to implement and test the algorithm of Yatziv, Bartesaghi and Shapiro and compare it to the original. [Here's my report](pdfs/LTTErikoistyo_Sariola.pdf), from 2007. Unfortunately, the report is in Finnish, but non-speakers can enjoy the pictures of shadows from point-like light sources.

Fast forward to today, I thought it would be fun to implement the algorithm on the C64, using the assembly language. The O(N) trick was taken further so that when accepting a cell, its accepted value was the rounded integer value. Furthermore, several smaller tricks were done to make it even faster:

* All math is 8-bit integers and solutions to the eikonal equation are found using table lookups.
* Priorities are never updated in the priority queue, because in 2D, usually the priority gets updated only once if at all. Instead, a cell can be added several (usually two) times to the priority queue and when considering a cell, we just check if it has been already accepted and skip it in the case it has. This saves enormous amount of memory because we don't need backpointers for the cells to update their priorities in the queue.
* When a cell is considered for the first time, a special temporary value is written to the output array. For example, if the cell considered was north, then a temporary value NORTH is written to the output array. Next time, when this cell is considered, if there is one of these temporary values, we know to which direction is the smallest cell and if the currently just-accepted cell is diagonal to that, we know which two cells should be used to solve the eikonal equation.
* The elements are not removed from the list one by one, but the smallest bin is freed at once. The whole bin is moved in the list of unused elements; this is just tying two linked lists together and very fast.
* Finally, code was carefully optimized by counting cycles in a debugger. The code was primarily optimized for speed, not size, but the algorithm is still only 512b.

Here's a demo of line of sight / shadow calculations, running on VICE.

![Line of sight / shadow calculations on C64](images/C64-fast-marching-los.png)

Sources and more explanations can be found from the [repository](https://github.com/vsariola/c64-fast-marching).