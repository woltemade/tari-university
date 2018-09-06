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

Formal requirements for a consensus protocol may include [[WPC]]:

* **Agreement**: All correct processes **must** agree on the same value.
* **Weak validity**: For each correct process, its output **must** be the input of _some_ correct process.
* **Strong validity**: If all correct processes receive the same input value, then they **must** all output that value.
* **Termination**: All processes **must** _eventually_ decide on an output value.

#### Some interesting proofs

For systems with _n_ processors, of which _f_ are Byzantine, it has been shown that _no algorithm exists_ 
that solves the consensus problem for _f > n/3_.[[ATT04]]



### Binary consensus

A special case of the consensus problem called binary consensus restricts the input and hence the output domain to a 
single binary digit {0,1}. When the input domain is large relative to the number of processes, for instance an input 
set of all the natural numbers, it can be shown that consensus is impossible in a synchronous message passing model [[WPC]].

### Synchrony

Many proposals make one or other assumption about _synchrony_.

_Synchronous_ protocols make the assumption that a message will be delivered within a certain time, ΔT.
                                                                                                   
In synchronous systems it is assumed that all communications proceed in rounds. In one round a process may send all 
the messages it requires while receiving all messages from other processes. In this manner no message from one round 
may influence any messages sent within the same round [[WPC]].

In a  _partially synchronous_ system, the system can alternate between good and bad periods of synchrony [[WPC]].

Asynchronous systems make no assumptions about when or whether messages arrive at their destination. Interstingly,
it has been proven that in a fully asynchronous, deterministic system _there is no consensus solution that can guarantee consensus
if even one node fails_ (the FLP theorem). This result does not state that consensus can never be reached: 
merely that under the model's assumptions, no algorithm can _always_ reach consensus in bounded time. 
In practice it is highly unlikely to occur. [[WPC]]

### Determistic protocols

For deterministic protocols, all honest nodes reach consensus by round _r_ for some a priori known constant _r_. 
For non-deterministic or probabilistic protocols, the probability that an honest node is undecided after _r_ rounds 
approaches zero as r approaches infinity. [[HN1]]

For synchronous protocols, roughly speaking, messages are guaranteed to be delivered after a certain bound ∆, but the asynchronous protocols don’t have such a bound.

## A brief survey of BFT-CMs

Many peer-to-peer online Real-time strategy games use a modified Lockstep protocol as a consensus protocol in order 
to manage game state between players in a game. Each game action results in a game state delta broadcast to all other 
players in the game along with a hash of the total game state. Each player validates the change by applying the delta 
to their own game state and comparing the game state hashes. If the hashes do not agree then a vote is cast, and those 
players whose game state is in the minority are disconnected and removed from the game (known as a desync.)[[WPC]]

## Paxos

The Paxos family of protocols includes a spectrum of trade-offs between the number of processors, number of message 
delays before learning the agreed value, the activity level of individual participants, number of messages sent, 
and types of failures. Although the FLP theorem says that there's no deterministic fault-tolerant consensus protocol 
that can guarantee progress in an asynchronous network, Paxos guarantees safety (consistency), and the conditions that 
could prevent it from making progress are difficult to provoke [[WPP]].

Paxos achieves consensus as long as there are f failures, where _f < (n-1)/2_. These failures cannot be Byzantine (otherwise
the BFT proof would be violated). Thus it is assumed that messages are never corrupted, and that nodes do not collude to
subvert the system.

Paxos proceeds through a set of negotiation rounds, with one node having 'Leadership' status. Progress will stall if the
leader becomes unreliable, until a new leader is elected, or if suddenly an old leader comes back online and a dispute
between two leader nodes arises.

### Raft

Raft is a consensus algorithm designed as an alternative to Paxos. It was meant to be more understandable than Paxos
 by means of separation of logic, but it is also formally proven safe and offers some additional features [[WPR]]. 

Raft achieves consensus via an elected leader. 

Each follower has a timeout in which it expects the heartbeat from the leader. It is thus a synchronous protocol.

If the leader fails, an election is held to find a new leader. This entails nodes nominating themselves on a first-come, 
first-served basis. Hung votes require the election to be scrapped and restarted.

This suggests that a high degree of cooperation is required by nodes and that malicious nodes could easily collude to 
disrupt a leader and then prevent a new leader from being elected.

Raft is a simple algorithm but is clearly unsuitable for consensus in cryptocurrency applications.

## Chandra-Toueg

The Chandra–Toueg consensus algorithm, published by Tushar Deepak Chandra and Sam Toueg in 1996, relies on a special
 node that acts as a _failure detector_. In essence, it pings other nodes to make sure they're still responsive.
 
 This implies that
 * The detector stays online.
 * The detector must somehow be made aware when new nodes join the network.
  
The algorithm itself is similar to the Paxos algorithm, which also relies on failure detectors and as such requires
_f < n/2_, where n is the total number of processes.


## HashGraph

The Hashgraph consensus algorithm [[SHG]], was released in 2016.  It claims Byzantine fault tolerance under complete 
asynchrony assumptions, no leaders, no round robin, no proof-of-work, and eventual consensus with probability one. 
And high speed in the absence of faults. 

It is based on the gossip protocol, which is a fairly efficient distribution strategy that entails nodes randomly 
sharing information with each other, similar to how human beings gossip with each other.
 
Nodes jointly build a hash graph reflecting all of the gossip events. This allows Byzantine agreement to be achieved 
through virtual voting. Alice does not send Bob a vote over the Internet. Instead, Bob calculates what vote Alice would 
have sent, based on his knowledge of what Alice knows. 

HashGraph uses digital signatures to prevent undetectable changes to transmitted messages.

HashGraph does not violate the FLP theorem, since it is _non-deterministic_.

The Hash graph has some similarities to a blockchain. To quote the white paper: "The hashgraph consensus algorithm is
equivalent to a block chain in which the 'chain' is constantly branching, without any pruning, where no blocks are ever 
stale, and where each miner is allowed to mine many new blocks per second, without proof-of-work" [[SHG]].

Because each node keeps track of the hash graph, there's no need to have voting rounds in HashGraph; each node already 
knows what all of its peers will vote for and thus consensus is reached purely by analyzing the graph. 


### The HashGraph Algorithm: The Gossip protocol

The gossip protocol works like this:

Alice selects a random peer node, say Bob, and sends him _everything she knows_. She then selects another random node
and repeats the process indefinitely.

Bob, on receiving Alice's information, marks this as a gossip event and fills in any gaps in his knowledge from Alice's
information. Once done, he continues gossiping with his updated informaiton.

In this way, information spreads throughout the network in an exponential fashion.

![Figure 1 - HashGraph](../assets/gossip.png 'The history of any gossip protocol can be represented by a directed graph, 
where each member is a column of vertices. Each transfer event is shown as a new vertex with two edges linking the 
immediately-preceding gossip events.')

The gossip history can be represented as a directed graph, as in Figure 1. 

HashGraph introduces a few important concepts that are used repeatedly in later BFT consensus algorithms: famous 
witnesses, and strongly seeing.

### Ancestors

If an event (_x1_) comes before another event (_x2_), and they are connected by a line; the older event is an _ancestor_ of that event.

If both events were created by the _same node_, then _x1_ is a _self-ancestor_ of _x2_. 

**Note**: The gossip protocol defines an event as being a (self-)ancestor of itself!

### Seeing

If an event _x1_ is an ancestor of _x2_, then we say that _x1_ **sees** _x2_ as long as the node is not aware of any
forks from _x2_.

So in the absence of forks, all events will _see_ all of their ancestors.


```
     +-----> y
     |
x +--+
     |
     |
     +-----> z
```

In the example above, x is an ancestor to both y and z. However, because there is no ancestor relationship between 
y and z, the _seeing_ condition fails, and so y cannot see x, and z cannot see x.

It may be the case that it takes time before nodes in the protocol detect the fork. For instance Bob may create z and y;
 but share z with Alice and y with Charlie. Both Alice and Charlie will eventually learn about the deception, but until
 that point, Alice will believe that y sees x, and Charlie will believe that z sees x.
 
 This is where the concept of _strongly seeing_ comes in. 


### Strongly seeing

If a node examines its hash graph and notices that an event z _sees_ an event x, and not only that, but it can draw
an ancestor relationship (usually via multiple routes) through a super-majority of peer nodes; then we say that according 
to this node, that z _strongly sees_ x.

## Consensus 

The HashGraph algorithm goes through a number of virtual voting rounds.

TODO: Elaborate

## Drawbacks

* HashGraph is patented [[HGP]].
* The HashGraph white paper assumes that _n_, the number of nodes in the network, is constant. In practice, _n_ can increase,
  but performance likely degrades badly as _n_ becomes large [[HN1]].
* HashGraph is probably not "fair" as claimed in their paper, with at least one attack being proposed [[GRA18]]

Baird tries to address some of these drawbacks in a follow-up paper [[BAI16]]
# References
1. Consensus mechanisms. Wikipedia. [[WPC]]
1. Attiya, Hagit (2004). Distributed Computing 2nd Ed. Wiley. pp. 101–103. [[ATT04]]
1. Paxos (Computer Science). Wikipedia. [[WPP]]
1. Raft (Computer Science). Wikipedia. [[WPR]]
1. Protocol for Asynchronous, Reliable, Secure and Efficient Consensus (PARSEC). Chevalier _et. al._, June 20, 2018 [[MSP]]
1. Project Spotlight: Maidsafe and PARSEC Part 1. Flatout Crypto. Jun 24, 2018. [[FCP1]]
1. The Swirlds Hashgraph consensus algorithm: Fair, fast, byzantine fault tolerance, Leemon Baird, May 31, 2016. SWIRLDS-TR-2016-01 [[SHG]]
1. Swirlds Intellectual Property. https://www.swirlds.com/ip/ [[HGP]]
1. Hashgraph: A Whitepaper Review, Michael Graczyk, https://medium.com/opentoken/hashgraph-a-whitepaper-review-f7dfe2b24647 [[GRA18]]
1. Swirlds and Sybil Attacks, L. Baird, Jun 6, 2016, http://www.swirlds.com/downloads/Swirlds-and-Sybil-Attacks.pdf [[BAI16]]
1. Demystifying Hashgraph: Benefits and Challenges, Yaoqi Jia, Nov 8, 2017. https://hackernoon.com/demystifying-hashgraph-benefits-and-challenges-d605e5c0cee5

[WPC]: https://en.wikipedia.org/wiki/Consensus_(computer_science) 'Wikipedia - Consensus mechanisms'
[WPP]: https://en.wikipedia.org/wiki/Paxos_(computer_science) 'Wikipedia - Paxos'
[WPR]: https://en.wikipedia.org/wiki/Raft_(computer_science) 'Wikipedia - Raft'
[ATT04]: https://www.amazon.com/Distributed-Computing-Fundamentals-Simulations-Advanced/dp/0471453242 'Attiya, Hagit (2004). Distributed Computing 2nd Ed. Wiley. pp. 101–103. ISBN 978-0-471-45324-6.'
[FCP1]: https://flatoutcrypto.com/home/maidsafeparsecexplanation 'Project Spotlight: Maidsafe and PARSEC Part 1'
[MSP]: ocs.maidsafe.net/Whitepapers/pdf/PARSEC.pdf 'PARSEC WhitePaper'
[SHG]: https://www.swirlds.com/downloads/SWIRLDS-TR-2016-01.pdf 'Hashgraph Whitepaper'
[HGP]: https://www.swirlds.com/ip/ 'Swirlds Intellectual Property'
[GRA18]: https://medium.com/opentoken/hashgraph-a-whitepaper-review-f7dfe2b24647 'Hashgraph: A Whitepaper Review'
[BAI16]: http://www.swirlds.com/downloads/Swirlds-and-Sybil-Attacks.pdf 'Swirlds and Sybil Attacks'
[HN1]: https://hackernoon.com/demystifying-hashgraph-benefits-and-challenges-d605e5c0cee5 'Demystifying HashGraph'