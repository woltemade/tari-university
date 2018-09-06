# Consensus mechanism comparison

Scope: We're looking at Byzantine Fault Tolerant consensus mechanisms.

Strictly speaking, Bitcoin is *NOT* a BFT-CM because there is _never_ finality in nitcoin ledgers; there is _always_ a 
chance (however small) that someone can 51% attack the network and rewrite the entire history. This is why Bitcoin
only needs 51% honest nodes to reach "consensus". It's a _probabilistic_ consensus, rather than deterministic.

So in this paper, we're going to look at some proposals for conventional BFT-CM that requires 67% honest nodes to
guarantee consensus. We will examine 

* the assumptions that each proposal makes (and how strong those assumptions are in
real-world adversarial environments), 
* the theoretical efficiency of each proposal, 
* whether there are any reference implementations,
* the degree of permissionless that is tolerated (e.g. fully permissioned nodes vs. digital signing of messages vs. totally open), and 
* whether these proposals could be used in Tari for a maintaining distributed digital asset state.

## Terminology

There are many confusing terms in the art.

### Consensus

Distributed agents (these could be computers, generals co-ordinating an attack, or sensors in a nuclear plant) 
that communicate via a network (be it digital, courier or mechanical) need to agree on facts in order to act
as a coordinated whole.

When *all* the (non-faulty) agents _agree on a given fact_, then we say that the network is in consensus.

### Binary consensus

A special case of the consensus problem called binary consensus restricts the input and hence the output domain to a 
single binary digit {0,1}. When the input domain is large relative to the number of processes, for instance an input 
set of all the natural numbers, it can be shown that consensus is impossible in a synchronous message passing model [WP1].

### Synchrony

Many proposals make one or other assumption about _synchrony_.

_Synchronous_ protocols make the assumption that a message will be delivered within a certain time, $$ \Delta T $$.



## A brief survey of BFT-CMs

### PBFT

# References
[WP1]

[WP1]: https://en.wikipedia.org/wiki/Consensus_(computer_science) 'title'

