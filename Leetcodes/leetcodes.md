---
id: d3ubbvopk8
title: "LeetCodes"
cover: "_assets/686b0ed43ffc788743c8609b91c56b5b.jpg"
cover_position: 50
sort: 1000
version: 1
Collection: Leetcodes
Problem: Two Set Problem
---

## Problem solving

### Two Sum Problem.


The two set problem follows a set of numbers, and a target number, The idea is to find out what two numbers in the given set of numbers sums up to the target number.


<callout>

Example.\
Assuming we are given a set of numbers **x =  \[2,7,5,14\],** and  a target number **T = 9.**\
then it is right to assume that:

</callout>

```math
T = a +b
```

Where:

- a = first number in the list.
- b =  second number.

Given this explanation we can also substitute out to be

```math
a = b - T
```

This means that when we take a number in the set as a first number **(a)** and subtract it from the Target, we get the second number **(b)**. Then we check if b exists in the list, if it does then we have  the two numbers : <b>a </b>and **b.**


So to solve this in  rust  we must get the group of numbers and iterate through them taking the first and performing the operation we discussed earlier
