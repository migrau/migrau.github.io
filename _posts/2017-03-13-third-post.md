---
layout: post
title: Multi-blast. Speed up a blast local alignment.
description: ""
modified: 2017-10-03
tags:
---

## Situation

Probably you use blast and sometimes you need to run it with a huge number of reads. Usually, it works in a sequentially way. There are different ways to improve the runtime and obtain more accurated results, i.e using parameteres like:

 
But the most efficient way is to split the query fasta file in as many pieces as you could run in parallel in your cluster/local cpu.

## Solution

Steps:
 