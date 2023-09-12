# Making And Solving Maze
Let's make some mazes! I'm thinking of mazes like this one, which is a rectangular grid of squares, with walls on some of the sides of squares, and openings on other sides. The goal is to get from the red arrow to the green arrow.
"![Wikipedia](https://upload.wikimedia.org/wikipedia/commons/thumb/8/88/Maze_simple.svg/475px-Maze_simple.svg.png)"

The two main constraints are that there should be a path from entrance to exit, and it should be *fun* to solve the maze with pencil, paper, and brain power—not too easy, but also not impossible.

As I think about how to model a maze on the computer, it seems like a graph is the right model: the nodes of the graph are the squares of the grid, and the edges of the graph are the openings between adjacent squares. So what properties of a graph make a good maze?

There must be a path from entrance to exit.
There must not be too many such paths; maybe it is best if there is only one.
Probably the graph should be singly connected—there shouldn't be islands of squares that are unreachable from the start. In fact, maybe we want exactly one path between any two squares.
The path should have many twists; it would be too easy if it was mostly straight.
I know that a tree has all these properties except the last one. So my goal has become: Superimpose a tree over the grid, covering every square, and make sure the paths are twisty. Here's how I'll do it:

Start with a grid with no edges (every square is surrounded by walls on all sides).
Add edges (that is, knock down walls) for the entrance at upper left and exit at lower right.
Place the root of the tree in some square.
Then repeat until the tree covers the whole grid:
Select some node already in the tree.
Randomly select a neighbor that hasn't been added to the tree yet.
Add an edge (knock down the wall) from the node to the neighbor.
In the example below, the root, A, has been placed in the upper-left corner, and two branches, A-B-C-D and A-b-c-d, have been randomly chosen (well, not actually random; they are starting to create the same maze as in the diagram above):

 o  o--o--o--o--o--o--o--o--o--o
 | A  b  c|  |  |  |  |  |  |  |
 o  o--o  o--o--o--o--o--o--o--o
 | B|  | d|  |  |  |  |  |  |  |
 o  o--o--o--o--o--o--o--o--o--o
 | C  D|  |  |  |  |  |  |  |  |
 o--o--o--o--o--o--o--o--o--o--o
 |  |  |  |  |  |  |  |  |  |  |
 o--o--o--o--o--o--o--o--o--o--o
 |  |  |  |  |  |  |  |  |  |  |
 o--o--o--o--o--o--o--o--o--o  o

Next I select node d and extend it to e (at which point there are no available neighbors, so e will not be selected in the future), and then I select D and extend from there all the way to N, at each step selecting the node I just added:

 o  o--o--o--o--o--o--o--o--o--o
 | A  b  c|  |  |  |  |  |  |  |
 o  o--o  o--o--o--o--o--o--o--o
 | B| e  d|  | N|  |  |  |  |  |
 o  o--o--o--o  o--o--o--o--o--o
 | C  D|  |  | M|  |  |  |  |  |
 o--o  o--o--o  o--o--o--o--o--o
 | F  E|  | K  L|  |  |  |  |  |
 o  o--o--o  o--o--o--o--o--o--o
 | G  H  I  J|  |  |  |  |  |  |
 o--o--o--o--o--o--o--o--o--o  o

Continue like this until every square in the grid has been added to the tree. At that point there will be a path from start to goal. Some walls will remain; some will be knocked down.

Making Random Trees
I'll make the following implementation choices:

A tree will be represented as a set of edges.
An Edge is a tuple of two nodes. Edges are bidirectional, so to avoid confusion we will always us the tuple that is in sorted order: always (A, B), never (B, A). The constructor edge enforces that.
A node in a tree can be anything: a number, a letter, ... In this notebook we will make trees where the nodes are squares in a grid, but the function random_tree accepts nodes of any type.
The algorithm for random_tree(nodes, neighbors, pop) works as follows:
The arguments are:
nodes: a collection of nodes.
neighbors: a function such that neighbors(node) returns a set of nodes.
pop: a function such that pop(frontier) removes and returns an element from frontier.
The function keeps track of three collections:
tree: a set of edges that constitutes the tree.
nodes: the set of nodes that have not yet been added to the tree, but will be.
frontier: a queue of nodes in the tree that are eligible to have an edge added.
On each iteration:
Use pop to pick a node from the frontier, and find the neighbors that are not already in the tree.
If there are any neighbors, randomly pick one (nbr), add edge(node, nbr) to tree, remove the neighbor from nodes, and keep both the node and the neighbor on the frontier. If there are no neighbors, drop the node from the frontier.
When no nodes remain, return tree.
import random
from collections import deque, namedtuple

Edge = tuple
Tree = set

def edge(A, B) -> Edge: return Edge(sorted([A, B]))

def random_tree(nodes, neighbors, pop=deque.pop) -> Tree:
    """Repeat: pop a node and add edge(node, nbr) until all nodes have been added to tree."""
    tree = Tree()
    nodes = set(nodes)
    root = nodes.pop()
    frontier = deque([root])
    while nodes:
        node = pop(frontier)
        nbrs = neighbors(node) & nodes
        if nbrs:
            nbr = random.choice(list(nbrs))
            tree.add(edge(node, nbr))
            nodes.remove(nbr)
            frontier.extend([node, nbr])
    return tree
# Making Random Mazes
Now let's use random_tree to implement random_maze. Basically, we just create a collection of (x, y) squares, pass these to random_tree, and let it do the work. Note that:

A Maze is a named tuple with three fields: the width and height of the grid, and a set of edges between squares.
A square is denoted by an (x, y) tuple of integer coordinates.
The function neighbors4(square) gives the four surrounding squares.

