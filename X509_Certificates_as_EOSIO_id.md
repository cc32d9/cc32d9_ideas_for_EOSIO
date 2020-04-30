# Using X.509 PKI certificates as EOSIO blockchain identity


## Introduction

Public key infrastructure is a set of standards that has been in use
for several decades and is adopted by many governments for digital
signatures. Businesses and private persons can obtain X.509 identity
certificates from local governments and use them to sign legal
documents.

In addition to government-issued certificates, independent certificate
authorities and software companies are issuing X.509 certificates to
their users. Also it is quite common that a VPN service is issuing
certificates for user authentication.

This document proposes utilizing the existing PKI certificates as
EOSIO blockchain identity, in order to remove the burden of managing
private keys from the user.


## Background

An X.509 identity certificate is generally the user's public key
signed by Certificate Authority (CA). The certificate includes also
informational fields, such as purpose of the certificate, person's
name and identity, CA name, and validity period.

When a new certificate needs to be created, the user-facing software
is generating a new keypair (a combination of private and public key),
creates a certificate signing request (CSR) and sends the CSR to the
CA. The CA is validating the user identity (this may be a manual
process with physical presence) and issuing the certificate.

The user software is storing the private key and the certificate
securely. Some operating systems, such as MS Windows, provide a secure
infrastructure for generating keypairs and storing the private keys
securely. In Windows, the private key can be marked as exportable or
non-exportable.

When the user needs to issue a digital signature, the user software
(such as MS Office or Adobe Acrobat) prompts the user to select from
known user certificates for which there is a private key. Then the
software is hashing the original document and generating a digital
signature of the hash. Depending on how the private key is stored, the
user may get a prompt to enter a password for accessing the key. So
far I could not find such an option in Windows, and MS Office creates
the signatures in one click.


## RSA certificates

Typically government-issued certificates use RSA-2048, or sometimes
RSA-1024 private keys and SHA-1 as hashing algorithm.

The essential property of RSA algorithm is that the same input and
private key will always produce the same signature. (Elliptic curve
signatures include a random parameter which produces a number of valid
signatures for the same input).

An RSA signature is equal in length to the modulo of the private key,
so the signature would comprise of 256 or 128 bytes for RSA-2048 or
RSA-1024, correspondingly.


## The method of generating an EOSIO private key from RSA certificate

A smart contract on the blockchain is storing supplementary
information for accounts utilizing this method. A smart contract table
consists of rows with the following fields:

* account name

* certificate fingerprint (up to 32 bytes)

* random seed (32 bytes)

* flags (uint16_t)

The end-user software (wallet) knows the fingerprints of available
certificates, and is able to look up in the smart contract and
identify which EOSIO accounts are managed by certificates of this
user.

The flags field indicates various options that determine the private
key generation. For the moment, only the lowest bit is utilized. If
it's set to 1, a user passphrase is needed for generating the key.

The wallet composes the input for signing with the certificate's
private key. It consists of the 32-byte seed and user passphrase if
the flags field indicates it.

The wallet requests the OS to produce a digital signature using
selected certificate.

The wallet takes SHA256 hash of the signature, and the resulting byte
string is used as a `secp256k1` private key. If it's not a valid key,
a new SHA256 hash is taken on these bytes, and the procedure is
repeated until a valid key is obtained. The probability of having an
invalid key is 2**-128, so this repetition would be used extremely
rarely.

So, the private key is generated in wallet memory, and can be
immediately used for signing transactions. Once the wallet is closed
or the user logs off, the memory range storing the private key must be
securely erased.


## Usage with EOSIO blockchain

The proposed method minimizes the user's effort in everyday operation,
because their computer has already an installed certificate, and many
users are already trained to use such certificates for electronic
signatures.

This method may replace "traditional" private keys, but it's more
practical to add the certificate-generated key as an additional means
of signing transactions, keeping the static private keys in a safe
place.

The wallet software needs to be able to modify the account permission
after generating the new key. So, it needs access to the static
private key in some form.

Also in hierarchical organizations, the user wallet may generate an
`updateauth` transaction, sign it with the user certificate, and send
to the central authority which has the power of changing the keys on
user accounts.

X.509 certificates have an expiration date, and government-issued
certificates have usually validity of one or two years. Therefore, the
wallet software needs to keep track of expiration dates and inform the
user that a new certificate will be needed soon. Once the new
certificate is in place, the wallet software can replace the key in
user account automatically.

A totally new and unexplored capability opens up in using the
government-issued certificates for blockchain identity. They would be
able to replace the expensive KYC process completely. Such usage needs
a thorough design effort, keeping in mind legal and privacy
implications.


## Security considerations

Windows and MacOS unlock the private key storage when the user logs
in. Further on, the user is not asked for a password when the private
key is used. Therefore a Trojan malware may get access to it and issue
digital signatures without user consent.

A passphrase would increase the security significantly, but reduce the
user convenience and productivity.

So, for public blockchains, key generation without a passphrase can
only be recommended for accounts that hold low commercial value. For
example, gaming or social network accounts.

Enterprise environments are normally more secure, and corporate
security policy would normally include a mandatory antivirus and
controlled internet access. Also the corporate IT may automatically
generate and install X.509 certificates. In such environments,
proposed method allows an automated and user-friendly integration of
user workspace with blockchain applications.


## Further work

This document is only presenting a rough idea of a new method, and
more work is required to make it usable:

* Peer review among the blockchain community;

* Defining the method of automatic discovery of the smart contract
  with supplementary data;

* Formalizing the method in a standard definition document;

* Reference open-source implementation for various OS platforms;

* Design and development of KYC replacement platform.








## Copyright and License

Copyright 2020 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




