# WAX Side Chains design

The design described in this document is applicable to any other
public EOSIO blockchain. But WAX is the first place where it's
actually needed.

## Current state of WAX blockchain

WAX is a public blockchain using a slightly modified EOSIO software
(RSA signature validation intrinsic is added for RNG
implementation). Due to growing popularity of NFT, easy on-boarding,
and low fees it has experienced an enormous growth in the course of
2021. As of August 2021, the total state database is over 32GB in
size. The network has over 2 million accounts, and several millions
NFT objects in circulation.

In addition to the size of the state memory, CPU and RAM costs have
grown dramatically.

One of significant growth factors is easy user on-boarding via WAX Web
Wallet. The users are authenticating via social networks, and WCW is
managing the keys for the users.

The public API infrastructure is heavily loaded with user traffic,
mostly related to AlienWorlds mining. This motivates the projects to
build and maintain their own API infrastructure.

The blockchain CPU resource is quite well congested, so the users and
dapps have to stake significant amounts of tokens to be able to
operate.


## Goals of a sidechain

* Transparency for the user: the WAX user should operate on the
  sidechain seamlessly, using the same private keys and account names.

* Predictable resource costs: WAX mainnet is growing fast and in an
  unpredictable way. It makes it difficult for a business to plan the
  costs and future growth. The sidechain should provide a clear and
  predictable cost structure. The sidechain design should also allow
  building chains that are dedicated for one business application.

* WAX as a primary currency: the sidechain services should be paid in
  WAX. This increases the market demand for the token and unifies the
  business landscape.

* Standalone and long term operation: the need for teleporting
  resources, such as fungible or non-fungible tokens, between the side
  and main chain should be minimal, if existing at all.

* Playground for alternative resource models: a sidechain can
  introduce a new way of charging for resources. For example,
  transaction fees could replace staking and RAM purchasing.

* Independent infrastructure budgeting: the sidechain may define its
  own way of defining the number of block producers, the method of
  selecting them, and rewards structure. Ideally, a sidechain should
  operate as a self-sustainable business.


## Accounts synchronization

The sidechain system contract should prevent normal accounts from
creating new accounts and changing their permissions. These operations
should only be allowed to the oracle accounts.

Several independent oracles are reading the account information
updates from WAX state history nodes and create the sidechain multisig
transactions that create new accounts or update the permissions on
existing ones. The goal is to reflect all WAX accounts on the
sidechain with identical permissions.

A sidechain may also choose to synchronize only a subset of WAX
accounts. For example, only owners of a certain NFT or fungible token
would be allowed to operate. Although this would save on system
resources, it reduces the transparency and makes uniform front-end
operation difficult.

Contracts and memory content should not be automatically mirrored to
the sidechain. The contract owners should buy the sidechain resources
and deploy their applications according to their needs.



## Billing and payments


Ideally, all payments for the sidechain service should be in WAX and
stay on the WAX blockchain, in order to avoid the teleporting
infrastructure and unnecessary custodial accounts.

Also it is preferable that the sidechain does not create a tradeable
or speculative token.


### Purchasing a resource

The WAX user that needs to purchase a resource on the sidechain
interacts with the payment gateway contract, such as [POS contract
made by me](https://github.com/cc32d9/eosio_point_of_sale/), and
receives a resource token that can only be spent on particular
sidechain resource. The user interacts with the resource management
contract and indicates which sidechain accounts should receive the
resources and their amounts.

Oracles on the sidechain detect the instruction to top-up a certain
sidechain account resource, and report back to WAX contract when the
operation succeeds and irreversible.

The resource token should never be transferable from the sidechain
back onto WAX blockchain.


### Infrastructure earnings

Infrastructure providers get paid from the resource management account
in WAX, according to their business agreement. It makes sense to
indicate several categories of infrastructure service, and bill them
separately:

* Block signing and chain security. The infrastructure providers do
  their best to distribute the block production servers geographically
  and place them in legally independent locations. High-volume chains
  require also that network latency is within certain limits, so in
  some cases the production servers should reside within the same
  continent, following major Internet backbone destinations. As I have
  shown [in an
  article](https://cc32d9.medium.com/running-a-small-eosio-network-22506468cd02),
  a chain of 5 producers tolerates an outage of only one of them. A
  network of 9 producers would still be alive if two producers go
  offline.

* Public p2p service. As it is essential to have p2p access as much
  distributed as block production, this service would normally be
  provided by every infrastructure team.

* Public EOSIO API service. A small blockchain does not necessarily
  need every BP to set up their own API service. It could be one or
  two instances with necessary level of redundancy.

* Public history service, such as Hyperion. History services require
  certain skills and amount of hardware, and it makes sense to
  motivate a skilled team with an adequate budget.

All in all, the network should provide a service that it can
afford.

Also the infrastructure providers want transparency and adequate
payment for their service, in a token that is easy to sell on the
market. Thus, having all payments in WAX is the easiest way to perform
the billing. As an alternative, a stablecoin on WAX could be chosen as
a means of payments for the sidechain resource.



## Resource billing models

Sidechains open up a larger field for experimenting and trying
different resource models. Here are some possible options:

### Traditional (EOS) resource model

User stakes the system token for CPU and NET, and buys RAM from Bancor
algorithm contract. Major drawbacks:

* The network does not earn from usage. Free transactions become a
  heavy burden for the infrastructure.

* Unpredictable CPU and RAM costs, which makes business planning
  difficult.

The benefit is that the technology is proven and not much needs to be
changed in vanilla system contracts.


### Transaction fees

There are several approaches to introducing transaction fees in an
EOSIO blockchain. To name a few,

* [FIO](https://github.com/fioprotocol) is running a heavily
  customized version of nodeos. Also it's an application-specific
  network, so only the system contracts are allowed. Each of system
  actions has an associated price factor, and the final price is
  defined by a median from prices suggested by block producers. Each
  action has also a spending limit, so that the user controls the
  costs on their side, disallowing the chain to charge more than
  expected.

* [Letting the contracts access the resource
  usage](https://github.com/EOSIO/eos/issues/9853). This option needs
  a feasibility research.

* [Setting the replenishing window to
  infinity](https://github.com/EOSIO/eos/issues/9376). The change
  itself is not difficult, but implications need to be researched.


### Flat rate

If a sidechain is dedicated to a particular application, the billing
can be as simple as a monthly invoice to the customer. The total bill
would be determined by current infrastructure costs, and the customer
would just order additional capacity if required.



## Governance

The difference of a sidechain from a public decentralized chain is
that it's dedicated to a particular business purpose. In such an
occasion, you would not expect anyone to register a BP and ask for
votes. Even more, voting is not needed at all. It's rather a business
agreement (or a legal contract) between infrastructure providers, and
between the network and its customers.

Such agreements should clearly document both the process of adding a
new infrastructure provider, removing it, or a way for the provider to
voluntarily leave the service.


## KYC and AML

One of potential applications for a sidechain could be a service that
is only available to KYC'ed accounts. But this would make the KYC flag
associated with WAX accounts publicly readable, which is a serious
privacy and security concern.

Probably the best case for a KYC'ed network would be to enforce
generation of random account names and using the keys that are not
known on any other public blockchain. Of course there would not be any
account synchronization with WAX mainnet, but the sidechain would
still be able to reuse the core components for gatewaying and billing.


## Architecture requirements

* Uniformity and reusability. The gateway contracts should be open for
  anyone to set up their own sidechain and billing workflow without
  having to copy and deploy them.

* Open source permissive licenses. The integration software, such as
  oracles and smart contracts, should be available for anyone without
  limits.






## Copyright and License

Copyright 2021 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
