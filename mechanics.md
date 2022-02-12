---
description: Learn exactly how the token and the NFTs interact with each other.
---

# Mechanics

### $LUCKY mechanics

When buying the $LUCKY token, there is a chance of receiving extra tokens as a reward. The default values are:

* 20% chance to receive 10% more
* 10% chance to receive 50% more
* 5% chance to receive 100% more

We can formulate this as a 3x2 luck matrix L:

```
L = [P R], where
P = [p, q, r] = [20, 10, 5]
R = [a, b, c] = [10, 50, 100]
```

We define the value V of L as

```
V = P • R = pa + qb + rc
```

V is measured in basis points, i.e. an index relative to 10000. Using the values of the above example we get **V = 20\*10 + 10\*50 + 5\*100 = 1200**. To understand what this means, think of it in percentage (divide by 100). **V = 12%** means that, statistically, a buyer will be rewarded 12% more tokens than they set out to buy.

Since this type of reward mechanic leads to inflation, deflationary mechanisms are put in place (burn on sell, "paying" for services with LUCKY, etc).

### Lucky Girls mechanics

Each NFT comes with a set of traits (also called attributes), each belonging to a specific category. The traits occur with varying frequency meaning some are rarer than others.

A **rarity score** is given to each Lucky Girl — it is the weighted combination of the rarity score of its traits. The weighting of each category is proportional to the average frequency with which traits appear in that category. The algorithm for calculating the rarity score of a Lucky Girl looks like:

```
// Definitions

CollectionCount[Trait] = Number of times trait appears in collection
NumTraits[Category]    = Total number of traits in category


// Calculations

Frequency[Trait] = CollectionCount[Trait]
                 / NumTraits[CategoryOf[Trait]]

AvgFreqencyOf[Category] = Sum(FrequencyOf[Trait] for every Trait)
                        / NumTraits[CategoryOf[Trait]]

RarityScore[Trait] = AvgFreqencyOf[CategoryOf[Trait]]
                   / FrequencyOf[Trait]

FinalRarityScore = Sum(RarityScore[Trait] for every Trait)
```

We denote the rarity score (i.e. FinalRarityScore above) for a Lucky Girl as **s\[TokenID]**, for instance s\[734].

### **Combined $LUCKY + NFT mechanics**

We define the sum **w** of the rarity score of each Lucky Girl held by an address:

```
w = Sum(s[TokenID] for every TokenID)
```

We then define the _compound rarity score_:

```
s(w) = a * w / (w + b)    where b > 0
```

Some notes on this definition:

* Note that **s(w) ∈ \[0, a]** and has the general shape of a curve starting at **0** and asymptotically approaching **a** as **w** approaches infinity.
* One can think of **b** as an incentive weight — the larger it is, the longer it takes for **s** to reach **a**. It should be chosen in proportion to the size of the rarity scores; if rarity scores (per token) range from 1 to x, a good value for b would be x/5 or x/4.
* In Solidity, to avoid rounding errors, it’s best to keep a large, e.g. a = 10000.

{% embed url="https://www.wolframalpha.com/input?i=discrete+plot+y+%3D+n%2F%28n%2B15%29+for+0+%3C%3D+n+%3C%3D+100" %}

We will now talk about the bonus matrix B, which is added to the luck matrix L in order to create a bonus reward for Lucky Girls holders. It can be parameterized by s in the following way:

```
B = | p_i(s)  r_i(s) |
    |  ...     ...   |
    |  ...     ...   |
```

The exact definition of each •(s)-mapping in **L** above is to be decided; at the very least, each should have an upper bound determined by **s**. A suggestion is

```
pi(s) = L_{i,p} * s * ɑ        (chance)
ri(s) = L_{i,r} * s * (1-ɑ)    (reward)

where ɑ ∈ [0, 1]
```

_When doing these operations in Solidity, keep in mind that alpha and s need to be scaled, e.g. with 10000, and operated upon in the right order as to avoid incorrect integer rounding._

So for instance, with **s = 0.5** and **ɑ = 0.2** we get

```
B = | 20ɑs   10(1-ɑ)s | = |   2    4 |
    | 10ɑs   50(1-ɑ)s |   |   1   20 |
    |  5ɑs  100(1-ɑ)s |   | 0.5   40 |
```

and, adding the bonus matrix to the example luck matrix above before applying the value function, we get

```
V(L+B) = 22*14 + 11*70 + 5.5*140 = 1848 = 18.48%
```

Reminder: V is defined in the $LUCKY mechanics section.

So, by owning an NFT with a compound rarity score of s = 0.5, a buyer would be increasing his expected $LUCKY reward from 12% to just above 18.48%.
