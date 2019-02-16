# Payout Service

This is a design draft for a universal payout service.


## Setting up a schedule

The payer needs to send tokens to payees based on a score or some
abstract counter of each payee. For example, this could be a total
achievement score at school, or contractors' billing hours, or a counter
of people passing by.

The payer registers a new payment schedule at the smart contract,
specifying the following attributes:

* Currency (EOS token name and token contract);

* Type of schedule (`hourly`, `counter`);

* Floating-point number, payment rate of tokens per unit.

The `regschedule` action creates a new schedule ID which is derived from
first 48 bits of the transaction ID.



## Hourly schedule

The payer sends `starthourly` action specifying the payee's account.

Later on, the payer sends periodically `paytime` action with an array of
payee accounts, and each payee receives the tokens according to the rate
and amount of time passed from the start or from previous `paytime`
execution.

The payer may send `delhourly` action with an array of payee account
names, and these accounts will be deleted from the schedule immediately
after sending the due amounts.



## Counter schedule

The payer sends `startcounter` action indicating the payee's account
name and the initial counter value.

Later on, the payer sends `updcounters` action specifying an array of
(payee, counter) tuples. Each payee receives tokens according to the
rate and the difference between counter values from prevuous payout.

The payer may issue `delcounters` action specifying an array of payee
accounts. The corresponding accounts will be deleted from the schedule.



## Handling exceptions

If the deposit is not enough, the `updcounters` or `paytime` transaction
does not fail, but the payer is notified about insufficient funds. The
payee will receive the tokens on next execution of `updcounters` or
`paytime`, provided that the deposit is sufficient.

`delhourly` and `delcounters` would fail if there's an outstanding
amount to be payed for a payee.

The counter is an unsigned 64-bit integer. Should `updcounters` reach
the maximum possible value for a 64-bit integer, the only way to
continue for the payee is to delete their account from the schedule and
start a new counter.











## Copyright and License

Copyright 2019 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
