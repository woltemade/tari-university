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

Formal requirements for a consensus protocol may include [WPC]:

* **Agreement**: All correct processes **must** agree on the same value.
* **Weak validity**: For each correct process, its output **must** be the input of _some_ correct process.
* **Strong validity**: If all correct processes receive the same input value, then they **must** all output that value.
* **Termination**: All processes **must** _eventually_ decide on an output value.

#### Some interesting proofs

For systems with _n_ processors, of which _f_ are Byzantine, it has been shown that _there exists no algorithm_ 
that solves the consensus problem for $f\gt \frac n3$.[ATT04]



### Binary consensus

A special case of the consensus problem called binary consensus restricts the input and hence the output domain to a 
single binary digit {0,1}. When the input domain is large relative to the number of processes, for instance an input 
set of all the natural numbers, it can be shown that consensus is impossible in a synchronous message passing model [WPC].

### Synchrony

Many proposals make one or other assumption about _synchrony_.

_Synchronous_ protocols make the assumption that a message will be delivered within a certain time, $\Delta T$.

In synchronous systems it is assumed that all communications proceed in rounds. In one round a process may send all 
the messages it requires while receiving all messages from other processes. In this manner no message from one round 
may influence any messages sent within the same round [WPC].

In a  _partially synchronous_ system, the system can alternate between good and bad periods of synchrony [WPC].

Asynchronous systems make no assumptions about when or whether messages arrive at their destination. Interstingly,
it has been proven that in a fully asynchronous system _there is no consensus solution that can guarantee consensus
if even one node fails_ (the FLP theorem). This result does not state that consensus can never be reached: 
merely that under the model's assumptions, no algorithm can _always_ reach consensus in bounded time. 
In practice it is highly unlikely to occur. [WPC]

## A brief survey of BFT-CMs

Many peer-to-peer online Real-time strategy games use a modified Lockstep protocol as a consensus protocol in order 
to manage game state between players in a game. Each game action results in a game state delta broadcast to all other 
players in the game along with a hash of the total game state. Each player validates the change by applying the delta 
to their own game state and comparing the game state hashes. If the hashes do not agree then a vote is cast, and those 
players whose game state is in the minority are disconnected and removed from the game (known as a desync.)[WPC]

### Paxos

The Paxos family of protocols includes a spectrum of trade-offs between the number of processors, number of message 
delays before learning the agreed value, the activity level of individual participants, number of messages sent, 
and types of failures. Although the FLP theorem says that there's no deterministic fault-tolerant consensus protocol 
that can guarantee progress in an asynchronous network, Paxos guarantees safety (consistency), and the conditions that 
could prevent it from making progress are difficult to provoke [WPP].

Paxos achieves consensus as long as there are f failures, where $f < (n-1)/2$. These failures cannot be Byzantine (otherwise
the BFT proof would be violated). Thus it is assumed that messages are never corrupted, and that nodes do not collude to
subvert the system.

Paxos proceeds through a set of negotiation rounds, with one node having 'Leadership' status. Progress will stall if the
leader becomes unreliable, until a new leader is elected, or if suddenly an old leader comes back online and a dispute
between two leader nodes arises.

### Raft

Raft is a consensus algorithm designed as an alternative to Paxos. It was meant to be more understandable than Paxos
 by means of separation of logic, but it is also formally proven safe and offers some additional features [WPR]. 

Raft achieves consensus via an elected leader. 

Each follower has a timeout in which it expects the heartbeat from the leader. It is thus a synchronous protocol.

If the leader fails, an election is held to find a new leader. This entails nodes nominating themselves on a first-come, 
first-served basis. Hung votes require the election to be scrapped and restarted.

This suggests that a high degree of cooperation is required by nodes and that malicious nodes could easily collude to 
disrupt a leader and then prevent a new leader from being elected.

Raft is a simple algorithm but is clearly unsuitable for consensus in cryptocurrency applications.
 
### PBFT

# References
1. Consensus mechanisms. Wikipedia. [WPC]
1. Attiya, Hagit (2004). Distributed Computing 2nd Ed. Wiley. pp. 101–103. [ATT04]
1. Paxos (Computer Science). Wikipedia. [WPP]
1. Raft (Computer Science). Wikipedia. [WPR]

[WPC]: https://en.wikipedia.org/wiki/Consensus_(computer_science) 'Wikipedia - Consensus mechanisms'
[WPP]: https://en.wikipedia.org/wiki/Paxos_(computer_science) 'Wikipedia - Paxos'
[WPR]: https://en.wikipedia.org/wiki/Raft_(computer_science) 'Wikipedia - Raft'
[ATT04]: https://www.amazon.com/Distributed-Computing-Fundamentals-Simulations-Advanced/dp/0471453242 'Attiya, Hagit (2004). Distributed Computing 2nd Ed. Wiley. pp. 101–103. ISBN 978-0-471-45324-6.'
