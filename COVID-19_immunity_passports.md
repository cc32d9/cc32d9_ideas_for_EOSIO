# COVID-19 Immunity Passports on EOSIO blockchain

## The problem

COVID-19 antibody tests are becoming available, and more people will
be known to be immune to the virus. Once the employers know the
immunity test results, they will be able to place their employees back
to normal working conditions, while keeping those not having the
immunity safe at home for remote work.

People will need the freedom to disclose their test results when they
need to. They will also need to protect their privacy and only
disclose this information at their will and discretion.

Once a person is willing to disclose the immunity information to an
employee or some other entity, there should be an easy and verifiable
way to do so.


## Data submission

Testing laboratories will submit the antibody test results to a public
EOSIO blockchain, such as Europechain. Each test result will include
the following information:

* type of the test (such as COVID-19 antibody test);

* code name of the laboratory (EOSIO account name);

* timestamp of the test (precision might be up to an hour or even a
  day, depending on established process in the lab);

* internal file number for auditing purposes;

* SHA256 of tested person credentials: full name concatenated with
  identity code, and spaces removed;

* 64-bit integer test result. In case of antibody test, the result may
  be 0 or 1;

The identity code is a string of symbols that is known to the tested
person and the lab. This could be a national social security number,
or an arbitrary set of symbols that the person decides to use. In case
of universal state provided identity number, the lab will not need to
negotiate it with the tested person, so the process would be simpler.

The blockchain smart contract organizes authentication, storage and
indexing of the data, so that it can be quickly retrieved knowing the
person's name and the identity code.


## Authorization

The smart contract must only allow the data submission by certified
laboratories. This will be achieved by using X.509 certificates, as
follows.

The smart contract is managed by a multi-signature authority that
delegates the management to a number of independent bodies, such as
blockchain block producers, or probably state authorities.

The administrators of the smart contract install X.509 certificates of
state authorities that are certifying the labs.

Each lab has a number of EOSIO accounts, and generates a set of
private keys that are solely used for data submission. The certifying
authority is issuing X.509 certificates signing the corresponding
public keys, and publishing them via the smart contract. The smart
contract is thus building a verifiable chain of trust, so that every
data submission key is known to belong to a certified laboratory.

Correspondingly, the smart contract is only accepting the data
submission signed by certified keys.


## Online test verification

Whenever a person wishes to disclose their test results to a third
party, they tell their full name and the identity code. The third
party performs a lookup in the blockchain smart contract table, and
validates the test result. Alongside the test result, authorization
information will be available for additional validation.


## QR code

In order to simplify the lookup of test information, the lab software
will generate a QR code that encrypts the person credentials hash with
a secret PIN. Such QR code would be printed on paper or stored in a
mobile phone.

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

* lab front end, so that it helps the lab publish test results on the
  blockchain. The lab software should be easy to integrate with
  existing systems, in order to minimize the processing overhead.

* end-user mobile applications for major mobile platforms. Probably an
  in-browser application would be suitable, but it needs to prevent
  data leakage and should not store any private data on web servers.


## Software re-usability

The software should be designed in a scalable way, so that multiple
countries would be able to use the same infrastructure without having
to duplicate the smart contracts and back ends.

Also it should be reusable for other kinds of tests, of medical or
other nature. For example, education or attestation certificates could
be managed in the same way.



## Copyright and License

Copyright 2020 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




