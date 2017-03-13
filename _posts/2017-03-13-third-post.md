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
 

The implementation is easier using workflows tools like [snakemake](https://snakemake.readthedocs.io/en/stable/) or [nextflow](https://www.nextflow.io/). Here there is a solution of snakemake, to use in a single cpu, multi-core. A version to use on a cluster is available on my [github](https://github.com/migrau).

 

