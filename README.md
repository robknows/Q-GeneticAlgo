# Q-GeneticAlgo

Some functions for defining and running genetic algorithms for fixed length binary
chromosomes. I've included an example usage of the functions.

## GA Operators

### Selection

Culls all chromosomes whose fitness is below the population median.

Potential improvements:

I am intending to make this configurable, such that you will be able to pass a
function which given a list of fitness results must return indices to those which
are intended to survive.

### Crossing

Given a list of chromosomes, pairs them up randomly and for each pair, chooses
a random point along the chromosome and swaps all genes up to that point between
the two chromosomes. Each pair's crossing returns two new chromosomes which are
then added to the working population.

Potential improvements:

This should become configurable, such that you can specify a function which given
two chromosomes will perform some crossing function between them and return a
list of offspring.

The pairing function could change such that more successful chromosomes get
paired more than less successful ones.

### Mutation

Flips a single bit within a given chromosome.

Potential improvements:

Flip multiple bits with some low probability.

## Operator wrappers

### Irradiation

Mutates a given chromosome with a probability given by a defined mutation-rate,
otherwise returns the chromosome as-is.

### Reproduction

For a given pair of chromosomes, crosses them with a probability defined by a
cross-rate, and if they are not crossed, then they reproduce by cloning. Cloned
offspring are guaranteed to have a mutation.

### Evolution

For a given generation, the following algorithm is applied:
1. Each chromosome is executed and its fitness assessed.
2. (Selection) Chromosomes with a fitness below the population median are culled.
3. (Crossing)  Survivors reproduce, mostly by crossing, sometimes by cloning.
4. (Mutation)  All chromosomes are irradiated, causing mutations.

The top level function, evolve, executes the above algorithm iteratively a given
number of times and returns the resulting chromosomes. If at some point the
number of survivors becomes 0 and the population is extinct, this is raised
as 'extinct.

## Usage

ga.q will handle the above issues, which are univeral to all genetic algorithm
problems of this kind, hence they can be refactored out into a library.

However, there are still a few things perculiar to the problem you are trying to
solve:

- Chromosome generation: You must define a method to produce your own fixed
length, random binary chromosomes for the initial population.

- Chromosome evaluation: You must define a function which takes as its only
argument a chromosome (as generated by you) and returns some kind of result that
you are looking for.

- Chromosome fitness scoring: You must define a function with the signiture:
scoreFitness:{[goal;attempt]...} where goal is some goal value and attempt is
some attempt at producing that goal. It must return a float as its given score.

By running ./build.sh the ga.q file will be moved into your QHOME (it does all the
appropriate checks first) so you can load it from any q program you're running by
doing `\l ga.q`.

## Example - exprFinder.q

The problem is to find a sequence of 7 symbols which, when parsed and
evaluated, produce some target number. Symbols can be 0-9 or +, -, *, %

If our target was 23, an example solution would be
"14+3\*03" = 14+3\*3 = 14+9 = 23.

If you look in exprFinder.q the required functions from the usage section above
are defined, as well as another function for tabulating the returned chromosomes
to help present more clearly the results.

In the example below I use these parameters:

- 0.7  is the crossing rate
- 0.01 is the mutation rate
- 20   is the maximum number of iterations/generations
- 1000 is the size of the initial population of chromosomes
- 45   is the target number

```
rob@computer:~/..../Q-GeneticAlgo$ q exprFinder.q 
KDB+ 3.5 2017.09.06 Copyright (C) 1993-2017 Kx Systems
l32/ cores()core ram rob computer 127.0.1.1 NONEXPIRE  

q)t:findExpr[0.7;0.01;20;1000;45]
q)t                               // Mostly looks very good
expression evaluation fitness
-----------------------------
"7+7--31"  45         0w     
"4+42+-1"  45         0w     
"55-6+04"  45         0w     
"42+2+1"   45         0w     
"42-0+-3"  45         0w     
"43+3-01"  45         0w     
"55-6+04"  45         0w     
"56-9+2"   45         0w     
"42+2+1"   45         0w     
"355-310"  45         0w     
..
q)`fitness xasc t                 // But there is some rubbish too
expression evaluation fitness 
------------------------------
"56-*9+2"                     
"4-+1-14"                     
"4+42++4"                     
"52-+2+1"                     
"3559+03"  3562       1.000284
"49-3310"  -3261      1.000302
"61-2301"  -2240      1.000438
"14-10-1"  5          1.025   
"4+42-41"  5          1.025   
"46+32+1"  79         1.029412
..
q)
```

And an example of what a random binary chromosome looks like:

```
q)randomChromosome[]
0 1 1 0 0 0 1 0 1 1 0 0 0 0 1 1 1 0 1 1 1 0 0 1 0 0 1 1
q)
```
