# Contracts

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
prevented by the on-chain *Validator*.

## Defining behaviour

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

*Validators* are covered in `validators.md` but `TxConstraints`
and `ScriptLookups` are new to us and off-chain. They are both
located in `Ledger.Constraints`.

## TxConstraints

`Ledger.Constraints` exports `TxConstraints i o`. `TxConstraints`
is a list of constraints on transactions. They are constructed by
the library functions listed below.

As can be seen by the above functions, `i :: RedeemerType a` and
`o :: DatumType a`, so when considering the library functions
listed below, remember: see `i`, think *redeemer*, see `o`, think
*datum*.

```haskell
mustPayToTheScript :: o -> Value -> TxConstraints i o

mustPayToPubKey :: PubKeyHash -> Value -> TxConstraints i o

mustMintCurrency ::
  MintingPolicyHash -> 
  TokenName -> 
  Integer -> 
  TxConstraints i o

mustMintCurrencyWithRedeemer :: 
  MintingPolicyHash ->
  Redeemer ->
  TokenName ->
  Integer ->
  TxConstraints i o 

mustMintValue :: Value -> TxConstraints i o 

mustMintValueWithRedeemer :: 
  Redeemer ->
  Value ->
  TxConstraints i o 

mustSpendAtLeast :: Value -> TxConstraints i o 

mustSpendPubKeyOutput :: TxOutRef -> TxConstraints i o 

mustSpendScriptOutput :: TxOutRef -> Redeemer -> TxConstraints i o 

mustValidateIn :: POSIXTimeRange -> TxConstraints i o 

mustBeSignedBy :: PubKeyHash -> TxConstraints i o 

mustIncludeDatum :: Datum -> TxConstraints i o 

mustPayToOtherScript :: 
  ValidatorHash ->
  Datum ->
  Value ->
  TxConstraints i o 

mustHashDatum :: DatumHash -> Datum -> TxConstraints i o 
```

## ScriptLookups

`ScriptLookups` is a data type that encapsulates information our
off-chain script will use that belongs *to other scripts*.

A value of type `ScriptLookups` is, similarly to `TxConstraints`,
constructed with the use of library functions included in
`Ledger.Constraints`:

```haskell
typedValidatorLookups :: TypedValidator a -> ScriptLookups a 

unspentOutputs :: Map TxOutRef ChainIndexTxOut -> ScriptLookups a 

mintingPolicy :: MintingPolicy -> ScriptLookups a 

otherScript :: Validator -> ScriptLookups a 

otherData :: Datum -> ScriptLookups a 

ownPubKeyHash :: PubKeyHash -> ScriptLookups a 

pubKey :: PubKey -> ScriptLookups a 
```
