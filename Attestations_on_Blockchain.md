# Attestations on EOSIO blockchain

## The problem

During their lives, people are receiving all kinds of certificates
issued by authorized entities, such as:

* education certificates;

* job qualification documents;

* vaccination certificates;

* antibody test certificates.

Certificate grantees have the freedom to disclose their papers to
discrete third parties when they wish to.

Those third parties have typically only limited means of verifying the
authenticity of provided certificates. Often it is an expensive and
slow process. In many cases fake certificates are fabricated in order
to gain certain benefits from presenting them.

Also verification of authenticity is sometimes compromising the
person's privacy: an organization that needs a proof of certificate
authenticity will have to inquiry its details from the certification
authority, thus disclosing the fact that a given person has
communicated with it.


## The approach

A public EOSIO blockchain, such as Europechain, Telos, EOS, WAX is a
universal storage of verifiable information, with important features,
such as:

* irreversibility: once submitted and passed the finality period, a
  transaction cannot be undone;

* authentication: every transaction needs to be signed by one or
  several private keys. There are no anonymous transactions on the
  blockchain.

* authorization: the hierarchical permissions system and smart
  contracts are verifying that only authorized accounts are performing
  transactions, according to the logic of a smart contract.

* multi-master replication: hundreds or thousands of blockchain nodes
  are synchronizing themselves with the blockchain and allow reading
  the state and submitting transactions in real time, with less than
  0.5s delay.

The approach is to use a public EOSIO blockchain for storing the
attestation data in a way described below. Such storage should address
the following requirements:

* privacy: no personal data should be placed openly on a
  blockchain. The lookup keys should be randomized in a secure
  fashion, so that it's too difficult or impossible to find out the
  real names or find certificates belonging to a person without their
  consent.

* verifiability: each certificate should be traced to an issuing
  organization, and the issuing organizations should be verifiable
  too.


## Data structures

### Organizations

A list of organizations needs to be maintained, with their full names
and addresses. The list includes authorities, such as government
departments, and certifying entities, such as universities or medical
labs.


### Types of papers

All certificate types are grouped and categorized, preferably using
international classification. Each type gets a unique number in the
system, and references to national or international classification.


### X.509 certificates of organizations

X.509 certificates are building a chain of trust: authorities publish
their public ECC keys in a verifiable manner, then they sign
underlying organizations' public keys and issue their corresponding
X.509 certificates.

As a result, each attestation published on the blockchain is signed by
a public ECC key that is known to belong to an authorized body.


### Delegations

A health department may only delegate medical tests to the test
laboratories, and those laboratories cannot issue job qualification
documents. So, the smart contract tables should define which types of
papers each certifying body is able to produce.



### Attestations

Certifying bodies and testing laboratories will submit their
attestations to the smart contract, specifying the following
information:


* type of certification paper

* organization issuing the paper (such as medical lab or testing
  center)

* timestamp of the test (precision might be up to an hour or even a
  day, depending on established process in the organization);

* internal file number for auditing purposes;

* SHA256 of tested person credentials: full name concatenated with
  identity code and type of certificate;

* 64-bit integer test or certification result. It could have only two
  possible values (true or false), or a more complex numerical
  structure, such as a score.

The identity code is a string of symbols that is known to the person
being certified and the certifying organization. This could be a
national social security number, or an arbitrary set of symbols that
the person decides to use. In case of universal state provided
identity number, the organization will not need to negotiate it with
the person, so the process would be simpler.

The blockchain smart contract organizes authentication, storage and
indexing of the data, so that it can be quickly retrieved knowing the
person's name, identity code and the type of certification paper.


## Authorization

The smart contract must only allow the data submission by authorized
bodies. This will be achieved by using X.509 certificates, as follows.

The smart contract is managed by a multi-signature authority that
delegates the management to a number of independent bodies, such as
blockchain block producers, or probably state authorities.

The administrators of the smart contract install X.509 certificates of
state authorities.

Each certifying body has a number of EOSIO accounts, and generates a
set of private keys that are solely used for data submission. The
certifying authority is issuing X.509 certificates signing the
corresponding public keys, and publishing them via the smart
contract. The smart contract is thus building a verifiable chain of
trust, so that every data submission key is known to belong to a
certified and authorized entity.

Correspondingly, the smart contract is only accepting the data
submission signed by certified keys.


## Online test verification

Whenever a person wishes to disclose their certificate to a third
party, they tell their full name, identity code and type of the
certificate. The third party performs a lookup in the blockchain smart
contract table, and validates the test result. Alongside the
certification result, authorization information will be available for
additional validation.


## QR code

In order to simplify the lookup of test information, the software at
the certifying body will generate a QR code that encrypts the person
credentials hash with a secret PIN. Such QR code would be printed on
paper or stored in a mobile phone.

So, the person would be able to show the QR code and enter the PIN, in
order to simplify the data lookup by the third party.


## Open source and open standard

It is vital that the data is managed in a fully transparent way. All
software components must be open sourced under a permissive license
and the data storage and retrieval should be well documented and
available for anyone.

A number of client applications needs to be developed:

* authority front end, so that it generates and places X.509
  certificates on the blockchain;

* certifying body front end, so that it helps it publish test results
  on the blockchain. The software should be easy to integrate with
  existing systems, in order to minimize the processing overhead.

* end-user mobile applications for major mobile platforms. Probably an
  in-browser application would be suitable, but it needs to prevent
  data leakage and should not store any private data on web servers.


## Software re-usability

The software should be designed in a scalable way, so that multiple
countries would be able to use the same infrastructure without having
to duplicate the smart contracts and back ends.




## Copyright and License

Copyright 2020 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




