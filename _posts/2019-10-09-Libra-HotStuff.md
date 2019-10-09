---
layout: post
title:  "FLP vs BFT, and how Libra deal with BFT Consensus"
date:   2019-10-09 15:59:53 -0700
categories: reading
tag: [distribute-system, crypto, consensus]
---

## Reading
First reading blog, sharing the reading and summary of Crash Fault-Tolerant protocal vs Byzantine Fault-Tolerance

A overall nice introduce artical for this topic [How Does Distributed Consensus Work](https://medium.com/s/story/lets-take-a-crack-at-understanding-distributed-consensus-dad23d0dc95)

### Crash Fault-Tolerant
Triditional Crash Fault-Tolerant protocal is used very common in daliy product. As the leader node is usually deployed by the service owner, 
Byzantine Fault-Tolerance does not need to be worried. A node is either working or crashed, it cannot be malicious (attacker).

Zab: Leader based; primary-backup;  
Usage: zookeeper

Paxos: Proposer/Acceptor/Learner; 3PC(more like 2PC + 2 Locking)

Raft: Leader based; Leader election; Synchrony Assumptions; Node Timeout;  
Usage: [Docker Swarm](https://docs.docker.com/engine/swarm/raft)

### Byzantine Fault-Tolerance

[DLS](https://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf)
- Introduced safety and liveness

[PBFT](http://pmg.csail.mit.edu/papers/osdi99.pdf)
- handle mostly (n - 1/ 3) faulty node
- leader exection
- random exection to change view

[Nakamoto Consensus](https://bitcoin.org/bitcoin.pdf), well, bitcoin
- effert selection
- reward base to keep liveness
- immtutable achieved with gossip protocal

### Thought on Libra
[The Libra Blockchain](https://cryptorating.eu/whitepapers/Libra/the-libra-blockchain.pdf)

The current Libra does not actually tell much about it IMO.
There are definitly no mining effort envolved in this.
Quote from the network section:
``` 
To join the inter-validator network, a validator must
authenticate using a network public key in the most recent validator set defined by the contract.
Bootstrapping a validator requires a list of seed peers, which first authenticate the joining validator
as an eligible member of the inter-validator network and then share their state with the new peer.
```

Also, the core Consensus Algorithm HotStuff is built based on PBFT, improved with network broadcast sctucture and leader election protocal. The is Byzantine Fault-Tolerance but on paper seems bad for scaling and too easy for public joining.

Gonna wait for more information.  
[Reference Reading](https://medium.com/ontologynetwork/hotstuff-the-consensus-protocol-behind-facebooks-librabft-a5503680b151)