# Crosschain Pegged Token

This is a concept for extending the liquidity of a token actoss multiple
EOSIO blockchains.


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

Each action on gateway contracts which is based on an event in
counterpart network is submitted by oracles. Each oracle account submits
the action independently from others. The gateway contract requires a
certain quorum of oracles before accepting an event. If any of oracles
is sending a different version of an event than others, the event is
blocked and escalated for human intervention.


## Pairing user accounts

The goal of pairing is to establish a link between two accounts in
different blockchains. It is performed by a 3-step handshake from each
side. The steps don;t need to be synchronized, and can happen with any
time gap between `pair` actions in both networks.

`telos:alice` and `mainnet:alice` are sending `pair` action to the
gateway contract, indicating the account name in the counterpart
network. The gateway contracts create pairing entries with status
"pending".

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
and that the transfer is in accepted currency. It creates an entry in
pending payments table.

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



## Incentives for oracles

`telos:gwrewards` is a smart contract that keeps track of rewards for
oracles that submitted `startxfer` and `okxfer` for successful
transactions. The rewards are calculated in TOKEN at a predefined rate,
proportional to the amount of transfer.

The issuer of TOKEN is responsible to pay the rewards to oracles. It
sends the token to `telos:gwrewards` and the contract sends it further
toward the oracles, proportionally to their due amount.

Once the due amount of rewards reaches a certain threshold, the oracles
are free to stop serving the TOKEN at their own discretion.





## Copyright and License

Copyright 2019 cc32d9@gmail.com

Ideas and contribution by Jesse Schulman (caleos.io)

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




