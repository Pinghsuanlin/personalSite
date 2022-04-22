# Association Rules: Market Basket Analysis

*Market Basket Analysis is one type of Association Rules. It's a data mining technique to find the optimal combination; to see which items go together to provide reccomendations, optimize product placement.*

## Measure Affinity
### Rule: A -> B
A implies B: If you want to know B, A is known. If A is true then B is true; if A is false then I dont make any claim about B whatsoever.

### Support: p(A&B): **Prevalence of an item set**
How likely is that we find the rule; How likely is that to find the subset of having both A and B.

### Confidence: p(B | A) = p(B&A) / p(A) : **Predictability of an association rule**
Probability of seeing B, given the existence of A.

#### Lift: p(B | A) / p(B) = p(A&B)/ ((pA) * p(B)): **Strength of association**
* Lift of 1 indicates independence (ie. no association)

For k items, number of rules is of the order 2^k.
We want to select rules indicating a **strong association (ie. high lift**) and **not rare (ie. high support)**.

Then we will set up rule filtering criteria
