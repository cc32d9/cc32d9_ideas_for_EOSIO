# Cross-blockchain token concept

This is a raw idea that needs further engineering.

Imagine a token that is circulating across several EOS blockchains, such
as EOS mainnet, TELOS, WORBLI, etc.

Each of networks runs a gateway contract which performs seamless
transfers of tokens between the networks. So, a token has the same
symbol and maximum supply, and the gateways make sure that the total
supply is the same and there is no double spending.

`telos:alice` is sending `telos:TOKEN@tokencontract1` to `worbli:bob`.

In Worbli network, this token is served by `worbli:tokencontract2`,
which belongs to the same physical entity as
`telos:TOKEN@tokencontract1`.

`telos:crosschaingw` and `worbli:crosschaingw` are two gateway contracts
that perform the transfers. The owner of the token contract registers
both contract names at the corresponding gateways, thus setting the
trust relationship.

Assuming that `telos:TOKEN@tokencontract1` existed already, and
`worbli:TOKEN@tokencontract2` is brand new in Worbli network, the owner
issues the whole maximum supply of `worbli:TOKEN@tokencontract2` to lock
in `worbli:crosschaingw` custody.

`telos:alice` performs the transfer in two stages: 1) it transfers the
desired amount of TOKEN to `telos:crosschaingw`; 2) it calls an
`trxxchain` action on `telos:crosschaingw` contract, specifying the
target network, target account, and the amount of assets, and this
action initiates a transfer to a Worbli recipient. Step 2 can transfer a
smaller amount than deposited.

This process needs some engineering effort: the back-end tool, probably
with the help of ZMQ plugin for `nodeos`, receives the event of
`trxxchain` action in Telos network, and initiates a transaction in
Worbli network. The `worbli:crosschaingw` contract should recognize this
as a valid and trustful transaction. Ideally, it would have a way to
verify Alice's signature, although Alice is not present at Worbli. This
needs a thorough design.

Also there can be several independent back-end nodes that process the
events, for redundancy. So, they need to detect that one of them has
already initiated a transfer in Worbli network, to avoid
duplication. Also the gateway contracts should detect duplications and
reject them.

Probably the backend should wait for the transactions to appear in
irreversible blocks, so the whole process may take several minutes.

Once the transfer is finished, `worbli:crosschaingw` makes a transfer of
TOKEN in Worbli network.


## Effort estimation

This project needs a working group and funding, probably from
BP. Suggestions are welcome.

Approximate resource estimation, in man-days:

1. Detailed workflow design and documentation: 5MD

2. Implementation of smart contracts and back-end tools: 10MD

3. Internal testing, then community testing: 5MD

4. Monitoring and reporting infrastructure: 5MD





## Copyright and License

Copyright 2018 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/




