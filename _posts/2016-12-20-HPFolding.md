---
layout: post
title: '2D HP folding problem: implementation of an exhaustive search algorithm.'
description: 'Implementation, in C++, of a backtracking algorithm that finds an optimal conformation of a protein in the HP model.'
tags: [algorithm, C++, optimisation, backtracking]
date-string: DECEMBER 20, 2016
image:
  feature: 2016-12-20/HPFolding_feature.png
external-links:
  github: tsucres/HPFoldingProblem
---

This project started as an assignment for the university. The task was to implement, in C++, a backtracking algorithm that solves the protein folding problem in the 2D HP model. 

The Hydrophobic-Polar model is a simplified model proposed by Ken A. Dill in 1985 [1] in which the proteins are represented by a sequence of two types of amino acids: Hydrophobic (H) and Polar (hydrophilic) (P). 

The 2D HP folding problem consists in finding an "optimal" conformation, in a 2D grid, of a protein represented by a sequence of 'H's and 'P's. 

The optimality of a conformation is defined by its score: the number of hydrophobic (H-H) bonds (number of H's that are neighbors on the grid but not in the chain). A conformation is optimal if there are no other conformation for the same protein with a higher score.

The following images show some examples of optimal conformations with their scores.


![ex1](https://github.com/tsucres/HPFoldingProblem/raw/master/screenshots/ex1_mini.png)
![ex2](https://github.com/tsucres/HPFoldingProblem/raw/master/screenshots/ex2_mini.png)
![ex3](https://github.com/tsucres/HPFoldingProblem/raw/master/screenshots/ex3_mini.png)


The HP Folding problem was proved to be NP-Complete in both its 2D and 3D versions [2]. 


I chose to base my work on [this paper](https://www.ncbi.nlm.nih.gov/pubmed/23899013) [3] which describes an exhaustive search algorithm with 6 pruning criteria. 

A more detailed explanation of the implementation is available on the [github repository](https://www.github.com/tsucres/HPFolding) of this project, along with benchmarks and build instructions.




### References

[1] Dill K.A. (1985). "Theory for the folding and stability of globular proteins". ([link](https://www.ncbi.nlm.nih.gov/pubmed/3986190))

[2] Bonnie Berger, Tom Leightont (1998). "Protein Folding in the Hydrophobic-Hydrophilic (HP) Model is NP-Complete". ([link](https://www.ncbi.nlm.nih.gov/pubmed/9541869))

[3] Giaquinta E., Pozzi L. "An effective exact algorithm and a new upper bound for the number of contacts in the hydrophobic-polar two-dimensional lattice model". ([link](https://www.ncbi.nlm.nih.gov/pubmed/23899013))
 