---
layout: post
title: Multi-blast. Speed up a blast local alignment
description: ""
modified: 2017-03-14
tags: [blast, alignments]
---

## Situation

Probably you use blast and sometimes you need to run it with a huge number of reads. Usually, it works in a sequentially way. There are different ways to improve the runtime and obtain more accurated results, i.e using parameteres like:

{% highlight python %}
-evalue -max_target_seqs -perc_identity 
{% endhighlight %}

or increasing the threads availables:

{% highlight python %}
-num_threads
{% endhighlight %}

But the most efficient way is to split the query fasta file in as many pieces as you could run in parallel in your cluster/local cpu.

## Solution

Steps:

{% highlight python %}
#split fasta query file
pyfasta split -n {blastJobs} {input.query};
#create database from the reference fasta file
makeblastdb -in {input.base} -dbtype {params.dbtype} -out blastemp/blastDB;
#blast each split query fasta
blastn -db blastemp/blastDB -query {input.fastas} -outfmt 6 -out {output} -num_threads {params.blasthreads}
#merge results
cat {input} > {output} 
{% endhighlight %}

The implementation is easier using workflows tools like [snakemake](https://snakemake.readthedocs.io/en/stable/) or [nextflow](https://www.nextflow.io/). Here there is a solution of snakemake, to use in a single cpu, multi-core. A version to use on a cluster is available on my [github](https://github.com/migrau).

{% highlight python %}
import subprocess,sys
from os.path import join,basename

# Globals ---------------------------------------------------------------------
#database fasta file
FASTA_DB = config["fadb"]
#query fasta file
FASTA_QUERY = config["faqy"]
#output prefix from query fasta file
prefix=basename(FASTA_QUERY).split(".")[0]
#nucl or prot
dbtype=config["dbtype"]
if dbtype=="nucl":
	rblast="blastn"
else:
	rblast="blastp"
#number of blast jobs to run (split the query.fasta file). 
blastJobs=5
#threads for each blast run.
blasthreads=12

SAMPLESX=[]
for i in range(0,blastJobs):
	SAMPLESX.append(prefix+'.%01d'%i)

rule all:
	input:
		prefix+'.blast.tsv'	

rule splitFASTA:
	input:
		base=FASTA_DB,
		query=FASTA_QUERY,
	output:
		trimmed=expand('blastemp/{sample}.fasta', sample=SAMPLESX),
		datab='blastemp/blastDB.nsq'
	params: 
		prefix=prefix,
		dbtype=dbtype
	shell:"""
		pyfasta split -n {blastJobs} {input.query};
		makeblastdb -in {input.base} -dbtype {params.dbtype} -out blastemp/blastDB;
		mv {params.prefix}.*.fasta blastemp/ && mv {params.prefix}.fasta.* blastemp/;
	"""

FASTA_DIR = 'blastemp/'
SAMPLES = glob_wildcards(join(FASTA_DIR, '{sample,[^/]+}.fasta'))
PATTERN = '{sample}.fasta'

rule blast:
	input:
		fastas=join(FASTA_DIR, PATTERN),
		datab='blastemp/blastDB.nsq'
	output:
		'blastemp/{sample}.tsv'
	params: 
		blasthreads=blasthreads,
		blast_type=rblast
	shell:"""
		{params.blast_type} -db blastemp/blastDB -query {input.fastas} -outfmt 6 -out {output} -num_threads {params.blasthreads}
	"""
#add -evalue 0.01

rule catblast:
	input:
		expand("blastemp/{sample}.tsv", sample=SAMPLESX)
	output:
		prefix+'.blast.tsv'
	shell:"""
		cat {input} > {output}
		rm -r blastemp/
	"""
{% endhighlight %}

