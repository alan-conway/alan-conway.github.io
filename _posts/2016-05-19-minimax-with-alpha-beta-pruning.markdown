---
layout: post
title:  "Minimax with alpha-beta pruning"
date:   2016-05-19 19:41:32
categories: posts
tags: ['#TDD', '#xUnit', '#Moq']
---
`#TDD #xUnit #Moq`

This posts continues the discussion of my reversi project. Source code can [be found here](https://github.com/alan-conway/Reversi) and if you'd like to download and play the game then you can do so by [clicking here](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release)

Here I discuss adding some AI into the game, in the form of the Minimax algorithm.

### _Minimax:_  
[Minimax](https://en.wikipedia.org/wiki/Minimax#Minimax_algorithm_with_alternate_moves) is an algorithm that is particularly well suited to two-player games, such as Reversi. Its goal is to evaluate the next moves available in a game to identify the best move to make.   
There are two main aspects of minimax - a heuristic and the algorithm itself.

#### _The Heuristic:_  
A heuristic is used to assign a value to an in-progress game. It should give a larger value for a game we are more likely to win, and a smaller value for one our opponent is more likely to win. We need a heuristic because the number of possible game will grow much too quickly to be able to examine them all, and we must limit the depth of our search.  
Since the goal of reversi is to occupy the most squares on the board at the end of the game, it's reasonable to think that a good heuristic would be to measure the number of squares we currently occupy, but this turns out to be a poor measure. The number of our pieces can change dramatically in just a few moves. Indeed it is often an advantage to have fewer pieces of our colour on the board until the endgame.  
There are a number of [better alternatives](https://en.wikipedia.org/wiki/Reversi#Strategic_elements) to choose from and I have opted to concentrate on corners and mobility:  
Corners are usually (but not strictly always) beneficial to capture since these are stable positions and cannot be recaptured by our opponent. A positive value is attributed when capturing a corner and a negative value when playing a piece in a square adjacent to an empty corner.
Mobility is the measure of how many squares we can legally play in. My heuristic adds positive value whenever the player has more options than their opponent, and negative when the opponent has more options.

#### _The minimax algorithm:_
The algorithm considers all possible moves up to a certain depth, recursively assigning each new game state a score. If we represent this as a tree structure then a leaf node is evaluated using the heuristic and other nodes are evaluated by taking either the maximum or the minimum of its child scores, depending on whether the node represents a move made by us or by our opponent.  

To illustrate the logic, suppose we only go so far as to explore our next possible move and look no further. Then we simply apply the heuristic to each choice and pick the move that yields the maximum score.  
If instead we also include all possible moves our opponent might make in response to ours, then the leaf nodes that we examine are the result of our opponents move. Our opponent will choose a move that benefits them the most, giving the smallest values according to our heuristic. So our best choice of move is the one that minimises our opponent's advantage.  
So as the algorithm considers moves at different depths, it alternately looks to maximise or minimise the score of the child nodes depending on which player is next to move. Accordingly, we think of the nodes at various depths of the tree as being min-nodes or max-nodes.

### _Alpha-Beta Pruning:_  
Even when we limit our exploration of moves to a certain maximum depth, the number of moves we need to look at grows very large. By using the technique of alpha-beta pruning, we can significantly reduce the number of steps in our calculation and arrive at our answer more quickly.  
The key to this technique is to record an upper and lower limit for each node in our tree, and to feed the results we find up to the parent node.  
Suppose, for example, that we are evaluating a particular node (N) in the tree, and that it is a max-node whose value is the maximum value of its child nodes. Suppose its child nodes are n1, n2, n3 and that we have already found that n1 has a value of x. Then, since N is a max-node, we can already say that the value of N must be _at least_ x.  
Since N is a max-node, we also know that its child nodes n1, n2, n3 are min-nodes. If we are currently evaluating all the children of n2, and find that one of these has a value that is less than x then we can deduce two things right away. Firstly, since n2 will take its value from the minimum of all its children, then it must necessarily have a value less than x (since n1 already has such a value). And secondly, since the value of n2 < n1, the value of n2 will not affect the overall value of its parent, N. Therefore, there is nothing to be gained by evaluating the other children and descendants of n2, and we can 'prune' n2, and its entire subtree.   
Following similar logic, by keeping track of the lower bounds (alpha) and upper bounds (beta) at each level, we can prune parts of the tree and speed up our overall calculation.

#### _Move Ordering:_
When using alpha-beta pruning, the order in which moves are evaluated can make a significant difference. Ideally we want to prune at the first opportunity to make the maximum saving.  
I have used a basic move ordering in which playing into a corner, if an available move, is considered first, and playing adjacent to an empty corner is considered last.  

### _Results:_
With the improvements described here, the result is a game that wins against me much more often than I win against it, so I consider this to be a success.  
The goal was not to produce an unbeatable game but simply to learn about the process of writing the AI logic, so I'm impressed that it's so strong.  
Why not [give it a try](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release) yourself?
