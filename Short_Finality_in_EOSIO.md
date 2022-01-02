# Short finality in EOSIO blockchain

In current EOSIO design, a block is declared as final after 2/3+1
block producers signed their blocks after it (it's a bit more complex
than that, see [Consensus
Protocol](https://developers.eos.io/welcome/latest/protocol-guides/consensus_protocol)
for details). This results in about 2 minutes finality in a network
with 21 active producers.

This document is an early design draft that proposes a consensus
algorithm with a much shorter finality (a matter of 1 second if no
network perturbation occurs).


## New p2p messages

The following new p2p protocol messages need to be defined:

* Block confirmation: an active block producer confirms the validity
  of a block, sending its number, hash, and a signature of the above.

* Block summary: an active block producer is sending a list of
  collected confirmations in a single message.

* Conflict signaling and resolution proposal: a block producer is
  indicating that there is a tie between several versions of a block,
  and suggests one version as the valid one.

* Recovery request, indicating that the block summary was not received
  within a defined timeout.


## Consensus protocol

Once a BP signs and broadcasts a new block, all other active producers
evaluate and confirm it, broadcasting the confirmation message. The
message is short, comparing to a typical block length, so it will not
generate too much traffic overhead.

1/3 of active producers that are next in the schedule after the
currently producing one are collecting the confirmation
messages. Whenever any of them collects 2/3+1 confirmations, it
broadcasts a block summary message. At this point, the block is
declared as irreversible.

Microforks are not uncommon in EOSIO networks, and they are usually
caused by double producing (the same BP having two servers signing the
blocks), or by a too long internet traversal time between the
producers, so that the last block in the schedule is overwritten by
the next producer.

In case of a microfork, multiple versions of a block with a given
number are broadcast throughout the p2p network. Whenever a block
producer receives a new version of an already seen block, it does not
issue a confirmation. But it stores the new version in a local cache.

If one of versions collected 2/3+1 confirmations, it is automatically
declared as final, and other versions are discarded.

It may happen that neither of versions has got enough confirmations
for a quorum. For example, in a network with 21 active producers, two
versions may collect 9 and 12 confirmations. In this case, either of
the next 1/3 producers in the schedule will issue a conflict signaling
message, indicating which version collected more confirmations. If
multiple versions are available with the same number of confirmations,
the numerically highest block hash is picked as a proposed candidate.

A hold-back time is defined, during which the confirmations are
accepted after the block was broadcast. After this timeout, the
conflict resolution proposal is issued even if some confirmations are
missing. For example, hold-back time is defined as 10 seconds, and one
out of 21 block producers is offline, and the network receives 10
confirmations for one block variant and 10 for the other. After 10
seconds of block signing, a conflict resolution is proposed, based on
collected confirmations.

Producers will receive the conflict signaling message and generate the
missing number of confirmations for the proposed block. Once 2/3+1
confirmations is available, the block becomes irreversible.

If an active producer does not receive a summary message within a
configured timeout (e.g. 10 seconds), it sends a Recovery request
message, indicating the block number that is missing the summary. All
other active producers are re-broadcasting their confirmations, and a
summary message is issued by any producer that has received 2/3+1
confirmations.


## Conclusion

It is possible to upgrade the EOSIO consensus protocol, so that the
last irreversible block is within 1 second from the current head block
in normal circumstances, and up to a few seconds in case of a network
outage or internet perturbation.






## Copyright and License

Copyright 2021 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
