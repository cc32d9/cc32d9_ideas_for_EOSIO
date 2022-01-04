# Short finality in EOSIO blockchain

In current EOSIO design, a block is declared as final after 2/3+1
block producers signed their blocks after it (it's a bit more complex
than that, see [Consensus
Protocol](https://developers.eos.io/welcome/latest/protocol-guides/consensus_protocol)
for details). This results in about 2 minutes finality in a network
with 21 active producers.

This document is an early design draft that proposes a consensus
algorithm with a much shorter finality (a matter of 1 second if no
network perturbation occurs), while keeping Byzantine fault tolerance
(surviving up to 1/3-1 of active block producers acting randomly or
abusively).


## New p2p messages

The following new p2p protocol messages need to be defined:

* Block confirmation: an active block producer confirms the validity
  of a block, sending its number, hash, and a signature of the above.

* Block finality: an active block producer is sending a list of
  collected confirmations in a single message. A copy of this message
  is also stored in blocks log.

* Conflict signaling and resolution proposal: a block producer is
  indicating that there is a tie between several versions of a block,
  and suggests one version as the valid one. The message contains the
  block number, copies of collected confirmations, and a proposal for
  the final block variant.


## Consensus protocol

Once a BP signs and broadcasts a new block, all other active producers
evaluate and confirm it, broadcasting the confirmation message. The
message is short, comparing to a typical block length, so it will not
generate too much traffic overhead.

1/3 of active producers that are next in the schedule after the
currently producing one are collecting the confirmation
messages. Whenever any of them collects 2/3+1 confirmations, it
broadcasts a block finality message. At this point, the block is
declared as irreversible.

Microforks are not uncommon in EOSIO networks, and they are usually
caused by double producing (the same BP having two servers signing the
blocks), or by a too long internet traversal time between the
producers, so that the last block in the schedule is overwritten by
the next producer.

In case of a microfork, multiple versions of a block with a given
number are broadcast throughout the p2p network.

Confirmation messages must be issued according to the following rules:

* if it's the first seen version of a block with specific number, the
  BP is sending a confirmation immediately.

* if it's a new version for a block that was already seen, and it's
  signed by the same producer, ignore the block and skip the
  confirmation.

* if it's a new version for an already seen block, but the signing
  producer is next down the schedule, send out a confirmation
  immediately. The previous confirmation becomes invalid, and all
  nodes must reject the old confirmation message if it arrives from a
  peer.

* a conflict signaling message resets the previous knowledge about
  confirmations for the given block. The node reacts this way on only
  the first received conflict signaling message, and ignores
  subsequent copies.

All p2p nodes in the network should validate if the received
confirmation adheres to these rules, and drop it if it violates the
rules.

If one of versions collected 2/3+1 confirmations, it is automatically
declared as final, and other versions are discarded. Every of 1/3
producers down the schedule broadcasts a finality message for this
block.

Every node in p2p network validates incoming finality messages: the
block hash must already be known, and there must be at least 2/3+1
signatures of active producers for the hash.

It may happen that neither of versions has got enough confirmations
for a quorum. For example, in a network with 21 active producers, two
versions may collect 9 and 12 confirmations. In this case, every
producer of the next 1/3 producers in the schedule will issue a
conflict signaling message, indicating which version collected more
confirmations. If multiple versions are available with the same number
of confirmations, the numerically highest block hash is picked as a
proposed candidate: bytes of the hash are compared as big-endian
256-bit integers, meaning the first bytes of the hashes are compared
first, and if they are equal, the second bytes are compared, and so
on.

A hold-back time is defined, during which the confirmations are
accepted after the block was broadcast. After this timeout, the
conflict resolution proposal is issued even if some confirmations are
missing. For example, hold-back time is defined as 3 seconds, and one
out of 21 block producers has not sent a confirmation, and the network
receives 10 confirmations for one block variant and 10 for the
other. After 3 seconds of block signing, a conflict resolution is
proposed, based on collected confirmations.

It may happen that a confirmation gets delayed because of network
congestion and geographical distance (or part of the network acting
maliciously), and reaches other block producers when a conflict
resolution message is already generated. In this case, if the conflict
resolution proposal conflicts with this late confirmation, the
proposal takes precedence.

Producers will receive the conflict signaling message and re-generate
the confirmations for the proposed block. Once 2/3+1 confirmations is
available, the block becomes irreversible.

The finality messages are stored by the nodes in their block
logs. Also whenever a remote peer is syncing against a local node,
the node must send the finality messages alongside the blocks as they
appear in the blocks log.

While finality message is being produced, the current active block
producer keeps signing new blocks, using the last block that it
confirmed as parent.

Every node in the network (both producing and non-producing nodes)
maintains its local chain of reversible blocks that stems from the
block that the node would confirm according to the confirmation
rules. The nodes also store local copies of fork variants without
evaluating them, until they can be discarded upon reaching the
finality. Once the parent block changes, the node re-evaluates the new
reversible chain by rolling back the state up to the grandparent of
the new parent.

The parent block may change as a consequence of the following events:

* next producer in schedule has signed a block with an already seen
  number;

* conflict resolution process has elected a block different from
  currently known parent.




## Conclusion

It is possible to upgrade the EOSIO consensus protocol, so that the
last irreversible block is within 1 second from the current head block
in normal circumstances, and up to a few seconds in case of a server
outage or internet connectivity problems.






## Copyright and License

Copyright 2021-2022 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
