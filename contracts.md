Contracts
=========

Plutus's `Contract` monad is used to define off-chain code, which
runs in one's wallet. It takes four parameters: `Contract w s e a
`, where:

`w` is a writer, which allows us to log messages.

`s` is the `Contract`'s "schema", which defines what its
capabilities are.

`e` defines the type of error messages produced by the `Contract`
n.b. `Text` is usually a good choice.

`a` is the result type of the computation n.b. consider `Maybe a`.

It is important to remember that anyone can make their own
off-chain interfaces, so any undesired behaviour should be
prevented by the on-chain _Validator_.

Defining behaviour
------------------

How a `Contract` should behave, is defined by a series of
constraints. Functions in `Plutus.Contract` to define a
`Contract`'s behaviour include:

```haskell
submitTxConstraints :: 
  TypedValidator a -> 
  TxConstraints (RedeemerType a) (DatumType a) ->
  Contract w s e Tx

submitTxConstraintsWith :: 
  ScriptLookups a ->
  TxConstraints (RedeemerType a) (DatumType a) ->
  Contract w s e Tx
```

_Validators_ are covered in `validators.md` but `TxConstraints`
and `ScriptLookups` are new to us and off-chain. They are both
located in `Ledger.Constraints`.

TxConstraints
-------------

`Ledger.Constraints` contains the definition of `TxConstraints i
o`. As can be seen by the above functions, typically `i ::
RedeemerType a` and `o :: DatumType a`, so when considering the
library functions listed below, remember: see `i`, think
_redeemer_, see `o`, think _datum_.

```haskell
mustPayToTheScript :: o -> Value -> TxConstraints i o

mustPayToPubKey :: PubKeyHash -> Value -> TxConstraints i o

mustMintCurrency ::
  MintingPolicyHash -> 
  TokenName -> 
  Integer -> 
  TxConstraints i o
```
