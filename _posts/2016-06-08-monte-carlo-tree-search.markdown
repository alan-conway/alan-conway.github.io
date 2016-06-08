---
layout: post
title:  "Monte carlo tree search"
date:   2016-06-08 12:04:12
categories: posts
tags: ['#TDD', '#Monte Carlo']
---
`#TDD #Monte Carlo`

This posts continues the discussion of my reversi project. Source code can [be found here](https://github.com/alan-conway/Reversi) and if you'd like to download and play the game then you can do so by [clicking here](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release)

Here I will discuss using monte carlo tree search as an AI algorithm in the game.  

### _Pure Monte Carlo:_  
Suppose we have a board with a number of possible moves for us to play next and we want to know which move to play. In a pure monte carlo approach, for each such move, we might choose to run 1000 game simulations and in each such game we would play out every move randomly until the game ends. We would note the outcome of the game and proceed with the next simulation. Once we have finished all simulations for all initial moves, we would look at which one led to the highest proportion of wins and that's the move we would choose.

### _Monte Carlo Tree Search:_
Monte carlo tree search works slightly differently to a pure monte carlo approach. Here instead we build up an asymmetrical tree, with more simulations being performed in areas of the tree that we deem to be more interesting. Once we've finished our simulations, the move that we pick is the one that we have simulated the most. I will explain more about this in a moment. But first lets go over the basics of the algorithm.

The four key parts to Monte Carlo Tree Search (MCTS) are:

- Selection
- Expansion
- Simulation
- Backpropagation


#### _Selection:_   
We start initially with a single node in our tree, which is our initial game state. Since there is only one node to select, the selection stage of the algorithm is trivial. We will come back to the Selection stage to explore how this works when there are more nodes in the tree.    

#### _Expansion:_  
Of all the possible child moves from the selected node, one is chosen at random and is added into to the tree. If all the child moves have already been added then there is no work to do here.  

#### _Simulation:_   
From there a simulation of the remainder of the game is run, with all the moves being chosen completely at random. We make a note in the node whether the outcome of the game was a win for black or a win for white, and we also keep a total of the number of games that have been played from that node.  

#### _Backpropagation:_  
Once a game has been completed, we propagate the results back up the tree to the node's parent and to all its ancestors. These all keep totals for the number of games played and the number of wins for black or white. So after each simulation, each ancestor increments its number of played games, and increments either the win count for white or black (or neither, in the case of a draw)  

#### _Repeat:_  
All of the above is repeated many times. The more repetitions, the better the overall results. The number of repetitions is usually determined by setting a fixed number, or by setting a fixed amount of time and performing repetitions until the time is up.  

![Monte Carlo Tree Search Image](https://upload.wikimedia.org/wikipedia/commons/b/b3/MCTS_%28English%29.svg)

### _Selection in more depth:_
Once there have been many iterations of the above steps, the tree will have grown (through the expansion step) to have a non-trivial number of nodes. How then do we select which node to expand and perform our simulations on?  
One point to make here is that we need to strike a balance between running simulations for nodes which we feel lead to beneficial results, and continuing to explore the rest of the tree in the search for even better results. The mathematics behind striking this balance is expressed in the form of a function known as UCT or the Upper Confidence Bound applied to Trees.  
The UCT is calculated according to this formula:		 ![UCT Formula](https://en.wikipedia.org/api/rest_v1/media/math/render/svg/4d380bf26dc9feb4d3cb45c58adb1027cd575479) 		in which:  
- _w<sub>i</sub>_ is the number of wins at the i-th node
- _n<sub>i</sub>_ is the number of games played at the i-th node
- _c_ is an exploration parameter - I've set this to √2 in my implementation
- _t_ is the number of games played at the parent node

To select a node on one of these iterations is to start at the root and if there are children still to be expanded then expand one of those as previously described, but when all children have already been expanded then navigate to a child node by selecting the child with the highest UCT value. And from there perform the same test recursively, which is to say either expand a child node if there is one, or navigate to the child with the highest UCT. And so on until we have either expanded some child or have reached a leaf node of the tree. From there we perform our simulation, backpropagate the results and repeat with another iteration.  

By using this selection method, we are preferring those nodes which lead to positive outcomes. Because of this, we expect to simulate favourable nodes more frequently than unfavourable nodes, and when we come to our decision about which next move to select, we can select the one that we have simulated the most because we know that it will be the most favourable.

### _Why Monte Carlo?_
There are advantages and disadvantages when it comes to using this monte carlo tree search (MCTS) approach with Reversi.  
One disadvantage is that it is generally slower than minimax although it can be shown that MCTS converges to minimax, so when the number of simulations is higher the strength of the algorithm is increasingly close to that of minimax.
But the interesting advantage of MCTS is that the algorithm can achieve this minimax-like strength without knowing anything about the strategy of the game. Unlike minimax which needs to use evaluation functions and heuristics to guide its decisions, MCTS can operate without them and can find its own interesting moves to make by letting them emerge from its random playouts.

### _My implementation:_
My application used a 1-9 strength rating for the AI algorithms it uses. For monte carlo, each setting represents a 0.5 second increase in thinking time from the previous setting, so setting the algorithm to level 5 means that it will search the tree for 2.5 seconds before making its move.  
Why not [give it a try](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release) yourself?

### _Further Reading:_
[https://en.wikipedia.org/wiki/Monte-Carlo_tree_search](https://en.wikipedia.org/wiki/Monte-Carlo_tree_search)
