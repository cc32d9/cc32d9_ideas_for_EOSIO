# Crosschain Pegged Token

This is a concept for extending the liquidity of a token actoss multiple
EOSIO blockchains.

## Comparing to BOS IBC

BOCORE team has released an inter-blockchain communication solution that
is designed to have seamless token transfers between two networks:
https://github.com/boscore/Documentation/tree/master/IBC

BOS IBC requires dedicated servers running nodeos software with the IBC
plugin, and these nodes are not suitable for anything else, like API
service. Our proposed solution needs nodeos servers with State History
Plugin enabled, and these servers can be used for other purposes.

BOS IBC requires a substantial amount of RAM in IBC contracts (15 or
more megabytes, which is about $3500 in current prices on EOS
mainnet. Our solution requires few hundred kilobytes of RAM for contract
deployment.

BOS IBC software is huge, and costly to maintain. A developer needs to
spend a significant amount of time in order to be able to support or
troubleshoot the software.

BOS IBC performs a deep verification of transaction eventts, whereas our
solution is based on a consensus of multiple independent oracles.

BOS IBC requires that the token has the same symbol in both
networks. Our solution does not have such limitation.



## Accounts and tokens

`telos:TOKEN@tokencontract1` is a token circulating its in native
blockchain.

`mainnet:TOKENPEG@tokencontract2` is a counterpart token in another
blockchain. It's pegged 1-to-1 to TOKEN and its whole supply is equal to
the amount of TOKEN deposited on the gateway contract. TOKENPEG is a
standard token contract as defined in `eosio.token` GitHub repository
and implements `retire` action.

`telos:mainnetgw` and `mainnet:telosgw` are a pair of gateway contracts
that operate together. Each instance is dedicated to working with only
one counterpart network and counterpart contract in it. The contracts
are fully symmetrical and run identical code.


`telos:alice` and `mainnet:alice` are a pair of accounts belonging to
the same user. They may or may not have identical public keys. They may
also have different account names.

`telos:bob` and `mainnet:chris` are arbitrary users in corresponding
networks.

`telos:oracle1`,... `telos:oracleN` and
`mainnet:oracle1`,... `mainnet:oracleN` are oracle accounts in both
networks. Server-side oracle tools are pushing transactions using these
accounts.


## Oracles and quorum

The solution relies on a number of independent oracles that process
events from the blockchain and submit the evidence to the gateway
contracts. Oracles are selected so that independent teams are operating
them. There are more oracles than it is required by quorum, and they
compete to submit their input as fast as possible, in order to earn the
rewards.

Each action on gateway contracts which is based on an event in
counterpart network is submitted by oracles. Each oracle account submits
the action independently from others. The gateway contract requires a
certain quorum of oracles before accepting an event. If any of oracles
is sending a different version of an event than others, the event is
blocked and escalated for human intervention.

A new permission called `oracle` is defined for oracles to send their
inputs. As the privates keys have to be stored on the servers, they
should be different from `active` and `owner` keys.

## Pairing user accounts

The goal of pairing is to establish a link between two accounts in
different blockchains. It is performed by a 3-step handshake from each
side. The steps don;t need to be synchronized, and can happen with any
time gap between `pair` actions in both networks.

`telos:alice` and `mainnet:alice` are sending `pair` action to the
gateway contract, indicating the account name in the counterpart
network. The gateway contracts create pairing entries with status
"pending". The pairing entries have also fields for keeping track of one
pending transfer: status flag, currency and amount. The user pays for
this RAM.

Oracles wait for these actions to appear in irreversible blocks in both
networks, and then send `ackpair` action indicating the pair of account
names and transaction ID in which the other side sent `pair` action.

The gateway contract verifies oracle inputs and changes the pair status
to "confirmed".

Oracles wait for the confirmed status to appear in irreversible block in
both networks, and send `finalpair` action to the counterpart contract,
indicating the pair of contract names and transaction ID of `ackpair`
action.

As soon as the gateway contract receives the quorum, it changes the
pairing entry status to "established".

Once the user decides to disconnect the pairing, she sends `unpair`
action to one of the sides. The gateway contract changes the entry
status to "unpaired", and oracles send `unpair` action to the gateway
contract on the other side.

Only token transfers from known accounts in "established" state are
accepted by gateway contract, and all other transfers are rejected.


## Opening accounts

The user needs to pay for RAM needed by token balance on each side of
the gateway.

The gateway contract keeps a record of which currencies Alice is willing
to transfer.

If `telos:alice` does not have TOKEN on her balance, she needs to send
`open` to the TOKEN contract or have someone transfer some amount of
TOKEN to her.

`mainnet:alice` needs to send `open` action to TOKENPEG contract.

`telos:alice` and `mainnet:alice` send `wantcurrency` action to the
gateway contracts indicating the corresondinhg token contract and
symbol. The gateway contract verifies that the they have a balance entry
in the token contracts.


## Initial state

Before any cross-chain transfers have happened, the `telos:mainnetgw`
has zero balance of TOKEN, and `mainnet:telosgw` has zero balance of
TOKENPEG.

Also total supply of TOKENPEG is zero.

Only `telos:mainnetgw` has the permission to `issue` the TOKENPEG.

Gateway contracts on both ends have a list of tokens they are accepting
in incoming transfers. Correspondingly, TOKEN is in the list of allowed
tokens in `telos:mainnetgw`, and TOKENPEG is correspondingly in the list
of allowed tokens at `mainnet:telosgw`. All incoming transfers in other
currency are rejected.


## Transfer toward pegged side

`telos:alice` has a positive balance of TOKEN due to some past
activity. She wants to move some amount to Mainnet.

She makes a normal `transfer` of TOKEN to `telos:mainnetgw` with
arbitrary memo string.

`telos:mainnetgw` verifies that Alice has "established" pairing status
and that the transfer is in accepted currency. It registers the payment
in Alice's pairing entry as a pending payment. Only one pending payment
is allowed for a user at a time.

Oracles wait for the transfer transaction to appear in irreversible
block, and send `startxfer` actions to `mainnet:telosgw`, indicating the
transfer arguments, transfer timestamp, and transaction ID.

`mainnet:telosgw` verifies that `mainnet:alice` has "established"
pairing status and that total supply of TOKENPEG plus transfer amount is
below the maximum supply.

Once the quorum of `startxfer` is achieved, `mainnet:telosgw` executes
two inline actions: first, it issues the exact amount of TOKENPEG to
itself, and then makes `transfer` of the total amount to
`mainnet:alice`, with the same memo as was specified in the initial
transfer by `telos:alice`.

If the quorum of `startxfer` is achieved later than 10 minutes after
inbound transfer, `mainnet:telosgw` rejects the action.

If outgoing transfer has not appeared in irreversible block within 10
minutes after the inbound transfer, the oracles send `failedxfer` to
`mainnet:telosgw`. In this case, `mainnet:telosgw` transfers the total
amount of TOKEN back to `telos:alice`.

If outbound transfer is seen in irreversible blocks within 10 minutes
after inbound transfer, oracles send `okxfer` to `mainnet:telosgw`. The
gateway contract deletes a pending transfer entry and sends a
notification to `telos:alice`. If Alice runs a smart contract that
rejects such notifications, later `cleanup` action will wipe the stalled
entries in pending transfers table.

Now `mainnet:alice` is free to send TOKENPEG to any account within
mainnet, such as `mainnet:chris`.

`telos:mainnetgw` keeps the amount of TOKEN on its balance. It is only
allowed to send out the token as a result of a cross-chain transfer in
the opposite direction.


## Transfer from pegged side

`mainnet:alice` has a positive balance TOKENPEG due to some past
activity. She wants to transfer some amount back to Telos network.

She performs a transfer of TOKENPEG to `mainnet:telosgw`, same way as in
the previous section.

The gateway contracts and oracles perform identical steps as in previous
section, with the following exceptions:

* upon receiving `startxfer`, `telos:mainnetgw` is making only a
  `transfer` of TOKEN, as it has a positive balance of it, and it is not
  allowed to issue TOKEN.

* upon receiving `okxfer` confirmation, `mainnet:telosgw` burns the
  TOKENPEG by calling `retire` action on its contract.



## Costs and rewards

The organization behind TOKEN is primarily interested in flawless work
of the gateway, and it is responsible for covering the operational
costs. The users do not pay any transaction fees.

The operational costs comprise of the following components:

* one-time RAM costs for deploying the contracts;

* support team developing the service, fixing problems, and interviening
  in case of oracles misagreements;

* Oracles hosting and operation.

The gateway maintainer defines a monthly budget to support the
infrastructure, and TOKEN organization is to support the operational
costs.

The payment for oracles is a fixed monthly amount distributed among
oracles proportionally to their successful transactions.


# Development costs

Development costs are estimated as follows:

* Programming and testing the gateway smart contract: US$3000.

* Programming the oracle software, based on Chronicle output: US$3000.

* Web UI programming: US$2000.

* End-to-end testing: US$2000.

Total budget: US$10,000.


## Copyright and License

Copyright 2019 cc32d9@gmail.com

Ideas and contribution by Jesse Schulman (caleos.io)

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




