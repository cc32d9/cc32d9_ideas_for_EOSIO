# Non-fungible Token for Dividends

This draft describes an idea of using NFT for tracking and paying
dividends or bonuses from EOS projects.

## Accounts and names

A project has its own `PTOKEN` issued by `ptcontract` smart
contract. The goal is to send dividends or bonuses to all `PTOKEN`
holders proportionally to their balances. As you will see in further
reading, the project contract does not necessarily have to be a token
contract, and `PTOKEN` itself is not taking part in the workflow.

The bonus payouts are made in `MONEY` token served by `mtcontract` token
contract (very likely, but not limited to EOS served by `eosio.token`).

`treasury` is an account within the project that is holding suffcicient
`MONEY` to pay out the dividends. This account is also running a smart
contract that automates the payouts.

`nftcontract` is the smart contract account serving the NFT tokens. The
token dedicated to the project is called `PNFT`.


## NFT properties

* `issuer`: the token contract account, such as `ptcontract`. This value
  is used as scope in accounts multi-index.

* `id`: a unique uint64 identifier for the token. When a new token is
  created, the ID equals to the first 64 bits of the transaction hash.

* `owner`: name of `PTOKEN` holder account.

* `due_amount`: `extended_asset` value indicating how much `MONEY` the
  owner is going to receive.

* `paid_amount`: `asset` value indicating total payouts in `MONEY` to
  the owner.


## Workflow

`nftcontract` is not transferring any `PTOKEN` or `MONEY`. It is only
serving to register the dividend amounts. We trust `treasury` to perform
the required transfers and to update corresponding fields in the NFT.

The NFT is transferable. There's not much sense in transferring it,
unless someone suggests a good use for it, but the `owner` can cnange by
`transfer` operation. Therefore, `due_amount` would be paid to the new
owner.


### Action: register

This action is called by `treasury` in order to register `ptcontract` in
`nftcontract`. The `ptcontract` must have a permission called `treasury`
delegated to the treasury account.

The action creates a new multi-index scope with a single NFT which has
`treasury` as the owner. Thus, the initial RAM expenses in creating a
multi-index are charged to the treasury account.

Arguments:

* `contract`: the name of the project token contract;

* `treasury`: the name of treasury account;

* `moneyctr`: payouts token contract;

* `moneysym`: payouts token symbol;


### Action: open

Once a `PTOKEN` holder wishes to receive the dividends, he or she
allocates a corresponding RAM amount by calling `open` action on
`nftcontract` with the following arguments:

* `owner`: the dividend receiver;

* `contract`: the project token contract, such as `ptcontract`


### Action: adddue

The treasury account calls this action to add to a token holder's due amount. Arguments:

* `owner`: the dividend receiver;

* `contract`: the project token contract;

* `amount`: the amount of payout;

* `moneyctr`: payouts token contract.
 

### Action: claimed

The owner sends `claim` action to `treasury`, requesting to transfer the
due amount of MONEY. The treasury account refers to the NFT accounts
table to determine the due amount, initiates the transfer, and calls an
inline `claimed` action on `nftcontract` with the following arguments:

* `id`: the identifier of NFT;

* `contract`: the the project token contract;

* `paidamount`: the amount of payout;

* `moneyctr`: payouts token contract.

The action updates the due and paid amounts in the NFT, accordingly.




## Copyright and License

Copyright 2018 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/

