# Enterprise Privacy on EOSIO blockchains (EPEBLO)

## The problem

Whenever businesses want to run their applications on a blockchain,
transaction privacy is one of their primary demands, and EOSIO
technology stack is not offering much to answer that. One may set up a
private EOSIO network using VPN and firewall policies, but it's by far
not sufficient to compete with such software suits as Hyperledger or
Corda.

A privacy design should address several key requirements:

* A private transaction should only be visible to members of a privacy
  group.

* A third-party observer should not be able to identify the senders
  and recipients of private transactions, nor their content.

* One participant of a blockchain should be able to transact with
  multiple privacy groups. A typical example is a manufacturer and a
  number of transport companies: the manufacturer should be able to
  place orders and track deliveries, but the transport companies are
  competitors and must not see the transactions of each other with
  this customer.

* All participants of a privacy group should have identical view of
  the blockchain state that is relevant to their group. Like in an
  open blockchain, every node has a synchronized state and information
  about all balances and other variables.




## The approach

Our proposal is to build a multi-layer topology, similar to that used in
VPN protocols:

* Control layer is responsible for establishing the privacy groups and
  communication withing them. A public EOSIO blockchain can be
  perfectly utilized for that.

* Transport layer is where the actual encrypted communication between
  parties is happening. It must not be based on a public blockchain,
  because one cannot delete data from a blockchain, so any security
  leakage will lead to a permanent disclosure of private
  communication.

* Private transaction layer. This is where smart contracts are
  executing and changing the state memory in transactional manner.

This document will not focus on fine details of the protocol, such as
byte format of the messages. Its main purpose is to describe the
concept and workflow, so that a detailed protocol would be implemented
later.




## Control layer

Few months ago we put together a small [demo called
"pobox"](https://github.com/cc32d9/pobox) in order to illustrate the
way to send encrypted messages between EOSIO blockchain accounts. All
encryption is done on the client side, and the blockchain is
responsible for communicating the recipient public key and storing the
encrypted message in smart contract memory.

The key feature of the demo (using ECDH with ephemeral sender key) is
that the sender's key is not involved. The message itself has nothing
to do with the sender's key, and therefore the sender is unable to
decrypt it. Only the recipient with their corresponding private key is
able to decrypt and verify the correctness of decryption by using the
MAC hash.

EPEBLO will use a similar approach, but with a difference:

* The recipient account is not specified when sending an encrypted
  message. 

* All potential recipients will listen to all encrypted messages and
  try decrypting them. Only those which match the MAC checksum will be
  accepted.

* The sender EOSIO account is a proxy, receiving encrypted messages
  from authenticated senders, and placing them on the blockchain.

The public blockchain is used as a transport for the EPEBLO control
layer, providing strict ordering, irreversibility, and automatic
synchronization between blockchain nodes.

The messaging smart contract assigns unique sequence numbers to the
messages and keeps them in memory until they expire.

The external observer would only see a sequence of cryptic messages,
impossible to associate with any blockchain player or any purpose.


### Message format

Each control message is a bytestring encrypted with `aes-256-cbc` with
a random initialization vector, where the encryption key is derived
from ECDH using the recipient's public key and a random one-time
private key.

The recipient public keys are known to the control layer software,
but are not necessarily used on the blockchain. For the sake of
privacy, it's better not to use these keys elsewhere.

Each message contains the following fields:

* Sender's public key. This key will be used by EPEBLO control layer
  for authentication and responding.

* Command code. This is an integer value determining the type of
  message. The protocol will define a number of request commands, and
  one acknowledgment command (ACK).

* Data length.

* Command data: the data content and format is determined by the command code.

* Nonce length and nonce content: a random number of random bytes for
  distinguishing between messages with the same content and for
  obfuscating the message type from external observer.

* ECC signature of SHA256 hash from all the fields above, made by the
  sender's private key.


Each request message is expected to be confirmed by recipient by
sending an ACK message back to the sender. The data field of the ACK
message is the SHA256 hash of the command fields, excluding the
signature. So, the sender needs to perform the hashing only once
before signing, and store it for reference until the acknowledgment
arrives.

A request can also be responded with NACK message indicating that the
recipient does not accept the request, and the NACK message contains
an error code and the original request's hash.


### Requests

* IDENTITY: Communicating parties need to identify themselves to each
  other and associate public keys with their names. The recipient of
  IDENTITY message is known by some means, such as X.509 certificate,
  or via paper communication. With this message, the sender provides
  its identity information, so that the recipient can associate their
  public key with a company or person name. The recipient will
  validate the identity by some offline means, and send ACK as a
  confirmation of verification process.

* NEW_PRIV_GROUP: a privacy group is created by its master, and this
  master defines the life cycle of the group. The message is sent by
  the master to all group members. If a new member needs to be added
  to an existing group, the same message is sent to the new
  member. The message data contains the following fields:

  * Group ID: a random 64-bit integer. The sender needs to make sure
    that the number is unique among known group IDs. If the recipient
    has already such a group ID in use, it should respond with a NACK
    message. Then the group master should generate a new proposal with
    a different group ID.

  * Transport cipher, such as `aes-256-cbc`.

  * Transport encryption key (in case of AES256, a 32-byte random
    number).

  * Encryption key lifetime: the date and time when the encryption key
    becomes invalid. The group master is expected to send a new key at
    least 1/3 of the lifetime period before expiration. Each
    encryption key has an identifier, which is equal to the first 4
    bytes of its SHA256 hash. The key identifier is used by transport
    layer in order to specify which of valid keys is used.
  
  * Transport type, such as IPFS, HTTPS, MQTT, ...

  * Transport parameters, such as server URL, so that communicating
    parties are able to send messages to each other.

  * EPEBLO blockchain genesis data. This is analogous to
    `genesis.json` file in EOSIO blockchains. The group master may
    attach the group to an existing EPEBLO blockchain, or start a new
    one.

* PRIV_GROUP_KEY_UPDATE: the message delivers a new transport
  encryption key from group master to its members. The old key remains
  still valid until its expiration time. The group members start
  using it only after they receive a transport message from the group
  master using this new key. The master will only start using it when
  all group members acknowledge the message.

* DESTROY_PRIV_GROUP: the master requests a termination of the group -
  for example, if a group member needs to be removed. In this case,
  the group is destroyed and a new one is created. Once the members
  acknowledge this message, they must not send any data within this
  group. The master must stop sending any data on the transport layer
  before sending this message.

* TRANSPORT_ERROR: any group member can send this message to the
  master, indicating that it's either unable to post new datagrams, or
  is missing datagrams expected by the transaction layer. In this
  case, the master destroys the group and tries to re-create it.

* RESEND_BLOCKS: if a new group member is attached to an existing
  EPEBLO blockchain, it requests the master to re-send all known
  blocks on the transport layer. Also in case of disaster, the group
  member may request a full replay.

* BLOCK_STATE: the master is periodically sending its current EPEBLO
  block number and ID, so that other group members can verify that
  their state information is synchronized. 

* CONSENSUS_ERROR: a group member receives a EPEBLO block and refuses
  to verify it. It sends this message to the master, and both master
  and the group member should stop processing further transactions and
  escalate the error to the operators.


## Transport layer

The transport layer can be implemented in a number of ways, and this
document is only specifying the functional requirements:

* All privacy group members should be able to post encrypted
  datagrams.

* All group members should be able to identify the datagrams belonging
  to their group, and read them in the same order as they were posted.

* The transport layer should guarantee the message delivery: if the
  posting operation succeeds, it means that other group members will
  be able to read it. The transport errors should be highly unlikely
  and only acceptable in disaster situations.

* The transport layer should discard old datagrams, either after all
  group members read them, or after a timeout. This is why a
  blockchain is not suitable as a transport layer.

* The transport layer should obfuscate the senders and receivers, and
  leave as little traces as possible (such as participant IP
  addresses).





## Transaction layer

We want to utilize as much as possible of EOSIO blockchain
functionality within the privacy group. But the standard EOSIO
software requires a low-latency P2P network and is sending speculative
transactions in real time, and signed blocks two times per second.

The transport layer may introduce delays up to dozens of seconds (for
example, IPFS may slow down the communication a lot). Also the default
EOSIO network protocol is pretty verbose, and the transport protocol
may simply not have such a capacity.

So, EPEBLO blockchain should be a derivative of EOSIO software, with
the following distinctive properties:

* Single BP. The transport layer is too slow to allow multiple block
  producers execute in schedule and confirm the irreversible
  block. So, a simple consensus is utilized: one block producer (most
  likely, the privacy group master) is forming blocks and broadcasting
  them to the rest of the group via the transport layer. Other group
  members are validating the blocks and report errors through the
  control layer.

* Blocks on demand. The BP is only forming a new block if there was a
  speculative transaction, and it waits 1 second for more speculative
  transactions to arrive before composing a block. The block becomes
  immediately irreversible and is broadcast to other group members
  via transport layer.

* Long living offline transactions. In EOSIO, a new transaction should
  refer to an irreversible block ID, and its expiration time is
  limited by 2^16 blocks, which is approximately 9 hours. Some
  communication media, such as low-orbit satellites or LoraWAN, may
  require longer offline periods. So, the new protocol should allow
  offline transactions to remain valid for at least 48 hours. Ideally,
  this maximum timeout should be configurable at the time of
  blockchain creation.

* Multi-chain architecture. A single node process (analog of `nodeos`
  in EOSIO) should support multiple independent blockchains. It should
  also permit runtime configuration, such as adding or removing new
  chains, or enabling the block production. The BP signing key should
  also be possible to be different for different chains.

* Multi-chain P2P protocol. In an enterprise configuration, one or
  several nodes would be communicating with the EPEBLO transport and
  control layers, and more nodes are needed inside the enterprise for
  sending transactions and querying the blockchain state. So, an
  internal p2P protocol should support multiple independent
  blockchains, and also auto-configuration: as soon as the nodes
  attached to the control layer receive a new blockchain creation
  request, other nodes in the enterprise should be able to
  auto-configure themselves for new chains.

* Multi-chain API. The HTTP API should be modified so that the request
  URL includes the chain ID. Access control to different chain IDs is
  left to the upper level proxies.





## Copyright and License

Copyright 2020 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




