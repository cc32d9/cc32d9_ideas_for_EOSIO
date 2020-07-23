# Anonymous Voting on EOSIO blockchain

## The problem

Voting is a fundamental mechanism of democracy, and up to now most of
it is happening on paper.

The need for online voting has been around for a long time, but it was
not as critical as in the era of a global pandemic. A solution for
online voting must address the following requirements:

* Proof of identity: the system should only allow eligible voters.

* Fully online: the voters should be able to vote using their usual
  Internet access terminals, such as personal mobile phones,
  computers, and public terminals.

* Offline option: the voters should still have a possibility to come
  up on-site and cast their votes.

* Each person should only be allowed to vote once.

* Full anonymity: nobody except the voter themselves should be able to
  know how anyone voted. Nobody should also know who has voted and who
  has not.

* Accountability: once the ballot is over, anyone must be able to
  verify the counts, using open source tools and open data, such as
  public blockchain.




## The approach

My approach is based on Elliptic Curve Cryptography (ECC), and
particularly on Elliptic-curve Diffie–Hellman key exchange
(ECDH). Some ideas are taken from my paper on [enterprise
privacy](Enterprise_Privacy_on_EOSIO.md) and the [P.O. Box
demo](https://github.com/cc32d9/pobox).

As demonstrated in pobox, a secret message can be generated in such a
way that only the owner of a private key can read and decrypt it. Also
only the owner of the key would know that the message is for them. In
pobox demo, both sender and recipient are indicated by their
blockchain account names, but it's unnecessary. Both sender and
recipient can be completely anonymous.


### Identity management

Each voter has a way to generate an ECC keypair securely, and store it
either in a hardware secure element (such as a smart card), or in a
secure enclave on a mobile phone.

While verifying the person's identity, the government issues a digital
certificate confirming the validity of the person's public key. The
certificate contains the following fields:

* unique serial number of the certificate;

* voting scope, such as the name of the city or organization;

* person's public key;

* validity period of the certificate;

* government signature of the above.

The certificate does not reveal the person's name. The government
needs to keep track of issued certificates securely and only a limited
circle of employees should have access to the association data between
real names and digital certificates.

The government is also maintaining a certificate revocation list, and
this list is immediately available for public reading. The best place
for such a list is a public EOSIO blockchain.

Upon expiration, a new certificate can be requested remotely, using
the person's private key.


### Infrastructure agents

During the ballot, two organizational units are established:

* *Vote Counter*. The role of Vote Counter is to generate its private
  key, publish the corresponding public key, and when the ballot
  finishes, publish the private key. As the vote counting key is
  published, anyone can re-count the votes by using public data.


* *Vote Validator*. The role of Vote Validator is to verify that a
  vote has been cast by an eligible person. The validator maintains
  its own private key which is never revealed. The validator should be
  independent from the local government.


### Voting process

Here by "user" we mean the client software on a mobile phone or in a
browser.

The user generates a random 256-bit number. We call it VoteID.

The user creates an encrypted message with an ephemeral (one-time
temporary) private key and Vote Counter's public key. The message
contains the VoteID and the vote information, such as names of
selected candidates.

The user creates another encrypted message with an ephemeral private
key and Vote Validator's public key, containing the following fields:

* VoteID

* copy of the government certificate

* voter's ECC signature of the above.

Both messages are sent to the blockchain through an anonymizing proxy,
so that the blockchain does not reveal the sender.

As the ballot is going on, Vote Validator decrypts the validation
messages, verifies the government certificates, and publishes approved
and disapproved VoteID's on the blockchain.

Upon each VoteID validation, Validator is creating an encrypted
message with its own private key and the voter's public key (this way
both can decrypt it) containing a confirmation of an accepted or
rejected vote.

The Validator maintains a database of certificate serial numbers and
VoteID's, preventing the same person from casting multiple votes. Such
a database can be organized on a public blockchain, using a SHA256
hash of a secret salt concatenated with the certificate number, and
used as a unique key in a smart contract table.

When the ballot ends, everyone waits for Vote Validator to finish the
validation process. There is no way to monitor the validation process
from outside, so the validator should indicate the end of the process
by sending a transaction to the blockchain.

Then, the Vote Counter private key is published on the blockchain. As
of this moment, anyone can decrypt the vote casting messages and count
the results.


### Disaster prevention

In order to prevent a loss of Counter and Validator keys, these keys
can be split among multiple independent parties, using Shamir's Secret
Sharing algorithm: it encodes the secret into a number of small
pieces, so that a smaller subset of those pieces can recreate the
original secret.

These pieces can be distributed securely among well-known entities,
such as lawyer offices.


### Offline voting

Many voters may not be able to use the online system, for a number of
reasons: they may not have time to obtain a digital certificate, or
may not be able to use a computer.

The offline procedure would look like following:

* The person approaches the polling place and presents their
  government ID.

* The official has a pile of pre-programmed smart cards containing
  private keys and government certificates. The certificates are
  listed in the revocation list as not activated.

* The official activates the smart card by submitting the association
  of the person's name with the certificate number into the
  government system. The official also submits a blockchain request
  to remove the certificate from the revocation list.

* The person receives the smart card and a PIN code, and proceeds to
  the voting booth.

* The voting booth runs the same software as used for online voting,
  and the further procedure is equivalent to the one described above.

* The person keeps the smart card securely, and it can be used again
  in next ballots without identity verification.
















## Copyright and License

Copyright 2020 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




