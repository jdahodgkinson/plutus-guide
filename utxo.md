# UTxOs

Unspent Transaction Outputs are the foundation of Cardano's eUTxO
(extended-UTxO) architecture.

eUTxOs carry *datums*, which are arbitrary pieces of data (such as
scripts), which are used to carry state information (such as the
UTxO's owner, or a valid time range in which it could be spent).

UTxOs are *immutable*, and when consumed must be consumed whole.

All UTxOs are is stable, quiet data. They will sit on the
blockchain indefinitely, until consumed. They are unable to act
autonomously, they *must* be consumed by a transaction to have any
effect on the wider blockchain.
