# Temporary transaction history in contract tables

This design was first implemented in my [POS
contract](https://github.com/cc32d9/eosio_point_of_sale) as of January
20th 2021. The purpose of this document is to explain the method and
to avoid any copyright claims.

## The problem

In EOSIO public blockchains, the finality period lasts approximately 3
minutes: once the transaction is signed and broadcast, it's included
in a block by a block producer, and later on, as 1+2/3 of block
producers have signed their blocks on top of the current one, the
block becomes irreversible.

There's a number of possible occasions for a transaction to be
dropped:

* HTTP API failure during the transaction submission;

* Poor p2p network connectivity;

* Congested queue of speculative transactions on BP nodes, resulting
  in expiration of the transaction;

* In case of a micro-fork, all transactions from affected blocks are
  re-evaluated and put in new blocks, with a new current
  timestamp. That may result in dropping or expiring the
  transaction. Micro-forks are typically happening because of poor
  time synchronization, network down-times, or wrong configuration on
  BP nodes, such as double production.


The blockchain projects need to react on transactions, such as
incoming payments. They should also make sure that the transaction
becomes irreversible, so that they can proceed with further
processing, such as shipping goods, or updating the user balance.

Up to now most of projects were relaying on history services, such as
Hyperion or dFuse, to track the reversibility of important
transactions. Any kind of outage in such services leads to service
delays, and potential money loss. A more reliable approach is to use
the EOSIO state history plugin and process its output, but that
requires dedicated infrastructure and increases operational costs,


The method that is described below allows a blockchain application to
be completely independent from history services.

The [POS contract](https://github.com/cc32d9/eosio_point_of_sale) uses
this method for processing incoming payments. But it can be applied to
outgoing payments or any kinds of important actions, such as NFT
transfers or minting of new assets.


## The method

The application runs one or several smart contracts for all important
operations, such as receiving or sending payments.

The smart contract keeps a log of transactions in its RAM tables. It
can be one table for all types of actions, or a separate table for
each operation. The contract designer needs to plan this in accordance
to the project goals.

A transaction log table contains the following fields:

* current timestamp (delivered by `current_time_point()` call);

* Optionally, transaction ID for better auditability. It's not used in
  most cases, but helps in debugging and adds a level of transparency
  for external observers;

* application-specific attributes, such as type of action, payer or
  recipient account, amount and currency, NFT identifier, transfer
  memo, etc. Anything that is important for the back-end to process.

The primary key in such a table depends on application specifics. It
cannot be the timestamp because the same block may contain several
such transactions. It can be the account ID or NFT ID if only one
action at a time is allowed for such an object. In many cases it would
simply be an auto-incremented integer.

The log table needs a secondary index on timestamp, for two purposes:

* The back-end may be designed to process the transactions in temporary
  order;

* The back-end will need to eventually clean-up old entries.

The log table may contain additional secondary indexes if the
application requires it.


In addition to the transaction log, it makes sense to keep the
timestamp or transaction hash of the latest update in a separate table
or a singleton. This will let the back-end detect the new transactions
with minimal overhead.

The back-end script communicates to the standard EOSIO API and is
monitoring the log tables, performing the following operations in a
periodic job, or a job that is triggered by new transactions:

* Retrieves the latest irreversible block (LIB) number with
  `/v1/chain/get_info` request;

* Retrieves the timestamp of LIB with `/v1/chain/get_block` request;

* Retrieves all transaction details from the log tables for entries
  that are not newer than the LIB timestamp;

* Processes them according to the application logic;

* Marks them as processed, either in an internal database, or in the
  smart contract.

* Periodically calls cleanup action that removes processes records
  from the contract RAM.


Once a new transaction is detected in the log, the back-end script can
check the current LIB timestamp and schedule further processing by
estimating when the current transaction becomes irreversible.

While processing the irreversible entries in transaction log, the
back-end may initiate new transactions and follow them up using the
same methodology.







## Copyright and License

Copyright 2021 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




