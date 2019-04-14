# Cross-blockchain token concept

This is a raw idea that needs further engineering.

Imagine a token that is circulating across several EOS blockchains, such
as EOS mainnet, TELOS, WORBLI, etc.

Each of networks runs a gateway contract which performs seamless
transfers of tokens between the networks. The token may or may not have
the same symbol in different networks, and the token issuer is
responsible for maintaining adequate supply in the networks. As they are
maintained by different contracts in different networks, we refer to
them as token pairs.

In addition to the gateway contract, there is a number of liquidity
providers. A liquidity provider has sufficient funds in both networks,
and it defines its own fees and service conditions.

One or multiple independent oracles are also set up. They work at a
fixed commission fee, and they can compete for submitting the
evidence. If multuple oracles send the same evidence, only the first
onne is accepted by the contract, and others are rejected.

Liquidity providers and oracles can be fully automated.

## Accounts and tokens

`telos:alice` is sending `telos:TOKEN@tokencontract1` to `worbli:bob`.

In Worbli network, this token is served by `worbli:tokencontract2`,
which belongs to the same physical entity as
`telos:TOKEN@tokencontract1`.

`telos:crosschaingw` and `worbli:crosschaingw` are two gateway contracts
that perform the transfers.

The liquidity provider has accounts `telos:chris` and `worbli:chris` in
both networks, holding enough quantity in both.

The oracle is operating under accounts `telos:daria` and `worbli:daria`,
correspondingly.


## Initiating a transfer

`telos:alice` sends an action `startxfer` to `telos:crosschaingw`,
indicating the following parameters:

* source account: `alice`;

* source token amount, symbol, token contract;

* memo string;

* destination network: `worbli`;

* destination account: `bob`;

* liquidity provider in Telos: `telos:chris`.

The transaction creates a Deal ID (`telos:deal_id`) which is an integer
formed by first 32 bits of the transaction ID. `telos:crosschaingw`
verifies that `telos:alice` has the required amount on her balance.

The liquidity provider detects the transfer request in Telos, and
initiates a transfer in Worbli. `worbli:chris` is submitting a
transaction `ackxfer` to `worbli:crosschaingw` with the following
parameters:

* source network: `telos`;

* source account: `alice`;

* deal ID in Telos network (`telos:deal_id`);

* token amount, symbol, token contract in Worbli;

* memo string;

* recipient account in Worbli: `bob`;

* liquidity provider in Worbli: `worbli:chris`.

The contract verifies that the liquidity provider has suficient balance
of the token. The transaction creates a new Deal ID (`worbli:deal_id`)
from 32 bits of transaction ID.

The liquidity provider detects the new deal in Worbli and sends a
confirmation action `confirmxfer` to `telos:crosschaingw`, with
parameters as follows:

* deal ID in this network (`telos:deal_id`);

* deal ID in other network (`worbli:deal_id`);

* token amount, symbol, token contract in Worbli;

* due date.

The contract notifies Alice that she can send in the token. If she is
not sending the token before due date, the deal is marked as expired and
eventually deleted by garbage collector.

`telos:alice` is performing a token transfer to `telos:crosschaingw`
with `telos:deal_id` in memo string. The contract verifies that the memo
string refers to a valid deal, and that the token amount is exactly the
same as specified in `startxfer`.

Chris detects that the deal on Telos is funded, and sends a token
transfer to `worbli:crosschaingw` with `worbli:deal_id` in memo
string. The contract verifies that the amount is exactly the same as
required. The contract transfers the token to `worbli:bob` immediately.

If `worbli:chris` fails to send the required amount by the due date, the
smart contract will reject any transfers for an expired deal. Once the
expiration date block becomes irreversible in Worbli, the oracle will
send `expired` action to `telos:crosschaingw`, and this action will send
the whole amount back to `telos:alice`.

If `worbli:chris` transfers the required amount successfully, the oracle
detects this event and sends a `success` action to
`telos:crosschaingw`. This action will release the token to
`telos:chris`, less the oracle fee. The corresponding fee will be sent
to the oracle account.


## Considerations

Multiple independent oracles are required to guarantee immediate
reaction. Probably a quorum of several oracles would be needed for deal
closing.



## Copyright and License

Copyright 2019 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




