# Universal EOS escrow contract

This is an idea for a future project. An EOS contract that will allow
setting up the deals between sponsors and contractors in any EOS
currency.

The main sponsor creates a deal with the following attributes:

* Currency contract and token name, and total amount
* Number of sponsors
* How many sponsors need to confirm the payouts
* Account name of contractor
* Account name of arbiter
* Short text memo for convenience
* Timeout for funding
* Timeout for the contractor to accept the deal
* Payout schedule, in percents (e.g. \[100\] or \[30 30 40\], etc.)
* Payout term (time period after claiming)

A new deal is created. Its ID is composed from first 64 bits of the
transaction ID.

Then, the main sponsor adds co-sponsor account names.

All sponsors place their deposits. The deposits are in equal parts, and
there's 1% commission on top of the total amount. The smart contract
accepts only the exact amount needed to fill up a co-sponsor's share,
and rejects all other payments. The payment has to contain a 64-bit
integer representing the deal ID.

The arbiter has to accept or reject the deal. The arbiter can set
automatic acceptance for deals with specific sponsor or contractor. Also
the arbiter can set up a blacklist and automatically reject deals with a
specific sponsor or contractor.

The 1% commission is split in halves between the escrow contract owner
and the arbiter. This amount is transferred to their accounts as soon as
the contractor accepts the deal.

If funding has not occurred, or contractor has not accepted the deal
before expiration, or the contractor rejects the deal, the funds are
returned to sponsors and the deal is erased.

Once the deal is fully funded, the contractor can accept the deal. The
contractor can also reject the deal at any time, except after the
acception.

Once the contractor performs the job, he or she claims a payout. If the
required number of sponsors signs off the acceptance, the payout is
automatically transferred to the contractor.

If the sponsors fail to sign-off the acceptance with9in the payout term,
the contractor can initiate an arbitration. The arbiter then decides if
the paymet has to be forced to release, or relased partially, or the
deal has to be canceled.

If the sponsors are not happy with the contractor's work, they also
initiate an arbitration. The arbiter decides if the deal has to be
canceled or partially resolved.

A website can collect the history of deals and each account's history,
thus building up their reputation scores. Also independent arbiters can
register themselves, with their respective skills, so that sponsors may
chose ones that fit their needs.




## Copyright and License

Copyright 2018 cc32d9@gmail.com

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
