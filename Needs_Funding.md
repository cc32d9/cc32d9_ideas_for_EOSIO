# Work that needs funding

2021-01-31 cc32d9@gmail.com


## EOSIO History Indexer

A POC is [described
here](https://cc32d9.medium.com/poc-for-eosio-history-indexer-eae200cdbf8f). The
goal is to build a new EOSIO history solution that would be more
efficient than Hyperion, especially in terms of storage and server
performance requirements.


## FONTO.io: NFC stickers for proof of authenticity

The project is briefly described at [FONTO.io](https://fonto.io/)
website. A working prototype for NXP NTAG213 NFC chips is
available. More work is required: developing the specs and software
for NXP NTAG 424 DNA chips, developing a mobile client, and also an
enterprise-grade labeling solution for mass production.


## EOSIO Smart Contract activity database

The project would deliver a public database service where daily
statistics of every smart contract are collected: its daily unique
users, counters of activity on every user and action, and aggregated
token transfers statistics. The data is being already collected on a
private server, but more work is needed in optimizing the data
collector for EOS Mainnet (performance is poor because of high
transaction volume on EOS), and funding is needed for hosting and
operations if this is provided as a public service.


## Electronic cash solution

A smart contract and its infrastructure that doesn't require users to
care about blockchain resources. The smart contract would take care of
keeping balances and verifying user signatures. Such a contract can
also interpret Bitcoin instructions and simulate a full Bitcoin
experience on a pegged token.


## Further improvements for Chronicle

[Chronicle](https://github.com/EOSChronicleProject) is a
community-sponsored project for interpreting the EOSIO state history
plugin output and presenting it as a stream of JSON objects. The
software is used in a number of infrastructure projects, and also some
exchanges are utilizing it for tracking incoming transfers.  The
software needs a number of updates, such as Webauthn key support,
compatibility with newest libraries delivered by Block One, and
possibility for non-JSON output, because JSON is quite CPU-intensive.


## IoT solution infrastructure

The project would deliver a set of protocols and reference
implementations for secure data delivery from IoT equipment, using
blockchain cryptography and EOSIO timestamping for integrity control.


## E-commerce software suits

Two components are already operational: [Payout
Engine](https://cc32d9.medium.com/eosio-payout-engine-a-reliable-way-to-automate-your-payments-fc1f1a523169)
and [Point-of-Sale
Contract](https://cc32d9.medium.com/eosio-pos-contract-enabling-e-commerce-for-everyone-6e72dd29864d).
They need also a whole set of surrounding infrastructure, such as
integration with popular e-commerce platforms, mobile wallets, and
ready-to-deploy web shop packages.


## Enterprise privacy solution for EOSIO

I wrote a [design draft](Enterprise_Privacy_on_EOSIO.md) for a
scalable enterprise-grade blockchain solution. The implementation
requires approximately 300 man-days.


## Software suite for exchanges

Exchanges are implementing their own solutions for accepting deposits
and sending withdrawals, and we've seen various issues with
them. There needs to be an open-source, validated software suite that
exchanges can take and integrate in their processes without having to
reinvent everything.


## Node planning and installation guides

A detailed guideline is needed for node operators, covering such
topics as functionality and performance planning, server resources,
and daily administrative tasks. It should also cover specifics of
biggest public chains, such as EOS or WAX. Also ready-made scenarios
that implement best practices proven by experienced engineers.



