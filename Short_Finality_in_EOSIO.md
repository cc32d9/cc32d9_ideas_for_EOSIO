# Short finality in EOSIO blockchain

In current EOSIO design, a block is declared as final after 2/3+1
block producers signed their blocks after it (it's a bit more complex
than that, see [Consensus
Protocol](https://developers.eos.io/welcome/latest/protocol-guides/consensus_protocol)
for details). This results in about 2 minutes finality in a network
with 21 active producers.

This document is an early design draft that proposes an implementation
of a new consensus algorithm with a much shorter finality (a matter of
approximately 1 second if no network perturbation occurs), while keeping Byzantine
fault tolerance (surviving up to 1/3-1 of active block producers
acting randomly or abusively).

ClarionOS team has published an [enhancement for the consensus
algorithm](https://edenos.notion.site/Fixing-Consensus-and-Improving-Irreversibility-44b3e92abeca4da0a07c0167287a6945),
and this document covers its application to EOSIO blockchain.


## New protocol message: Block confirmation

A new message type is defined: Block confirmation. An active block
producer confirms validity of a block, sending its height, hash,
confirming producer account name and a signature of the above.

Upon receiving the message via p2p interface, every node verifies its
validity:

 * signor is an active block producer;
 * signor hasn't confirmed this block height previously;
 * the signature matches the producer's block signing key.

Once the message is received and validated, it's stored in the blocks
log alongside the blocks. This needs a change in blocks log format.

Each node keeps all recent confirmations in memory until the
corresponding blocks are recognized as final.

A block signed is treated as an implicit confirmation by the block
signor.


## Consensus protocol

An active BP signs and broadcasts a new block. Other active producers,
once they receive it, validate it and proceed as follows:

* if it's the first occurrence of a block at this height, broadcast a
  confirmation message.

* if a block at this height is already seen, adopt it as the current
  head block, but do not send any confirmation. The previous version
  of a block is stored in a temporary cache.

Once a block X has got 2/3+1 confirmations and one of its successive
blocks Y has 2/3+1 confirmations, the block X is assumed to be final.


## Microforks handling

Microforks are typically happening as a result of either an error in
configuration that leads to double producing, or because of internet
congestion and late arrival of the last block to the next producer in
schedule.

Once a node sees a new version of a block at specific height, and if
the block has a valid chain of parents toward the last irreversible
block, the node switches to this new fork. The old version is kept in
the cache, as there might be a consensus condition that endorses the
older version.

Once a node sees that a block in an alternative fork has reached the
finality, it rolls back to up the common parent and adopts the final
branch, down to the last known head block.


## Conclusion

With the proposed protocol upgrade, an EOSIO blockchain can reach
sub-second finality if no microforks occur, and the network is fast
and reliable: once the block is signed, confirmations would arrive
within 200ms, and one more block is needed with confirmations. So,
700ms finality is achievable in good internet conditions.

If a small microfork occurs (1-2 blocks deep), the finality is delayed
for about a second.

One interesting side effect of the new protocol is that there can be
much more than 21 active producer, still resulting in finality within
a few seconds. The finality is only limited by internet throughput and
available processing power on the nodes.

The software needs a change in blocks log format, which means a need
for a full replay before the blockchain can switch to the new
protocol. Upgraded nodes can co-exist in the old blockchain, as the
finality algorithm is the same as before in the absence of block
confirmations.




## Copyright and License

Copyright 2022 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
