# Validators

In order for a UTxO to be spent, it must be consumed by a
*transaction*. For a transaction to take place, it is necessary
for it to pass some form of *validation*, otherwise its behaviour
is not defined.

Validation can work with three pieces of information:

1.  the *datum* of the UTxO it wishes to spend.
2.  a *redeemer* script, which the output spender defines.
3.  the *context* of the transaction.

## Contexts

Contexts are a key differentiator of Cardano to other blockchains,
such as Bitcoin and Etherium. On Bitcoin, Validators may only
access data living on a transaction and the redeemer of the input.
With Etherium, Validators may call upon information from *the
entire blockchain*. Cardano sits in the middle of these two
approaches, allowing a validator to see all of the inputs and
outputs of its transaction, but nothing outside of it.

## (Un)typed

The default Haskell interface for defining datums, redeemers and
contexts is quite low-level: a single `Data` type. We love Haskell
because type-driven development is the best. Plutus *does* provide
us with a method of defining our own datums and redeemers, as well
as providing a type for the context. We will omit any examples
using "un-typed" arguments.

The type signature of a generic, typed validator could be:

```haskell
myValidator :: MyDatum -> MyRedeemer -> ScriptContext -> Bool
```

If the validator returns `True`, then the transaction is
permitted. The type representing the context is `ScriptContext`:

```haskell
data ScriptContext = ScriptContext 
  { scriptContextTxInfo :: TxInfo
  , scriptContextPurposse :: ScriptPurpose
  }
```

The first record in `ScriptContext` is one that we will be using
often:

```haskell
data TxInfo = TxInfo
    { txInfoInputs      :: [TxInInfo] 
    , txInfoOutputs     :: [TxOut] 
    , txInfoFee         :: Value 
    , txInfoMint        :: Value
    , txInfoDCert       :: [DCert]
    , txInfoWdrl        :: [(StakingCredential, Integer)] 
    , txInfoValidRange  :: POSIXTimeRange 
    , txInfoSignatories :: [PubKeyHash] 
    , txInfoData        :: [(DatumHash, Datum)]
    , txInfoId          :: TxId
    }
```

So, when writing validators, it is common to call the appropriate
function to retrieve the `TxInfo` from the `ScriptContext`:

```haskell
myValidator :: MyDatum -> MyRedeemer -> ScriptContext -> Bool 
myValidator d r ctx = undefined 
  where 
    txInfo :: TxInfo 
    txInfo = scriptContextTxInfo ctx
```

As you can see from its definition `TxInfo` contains a tonne of
useful information about the transaction, which can be invaluable
for deciding if we want our validator to pass or fail.

## Compiling to Plutus Core

As the validators we write are going to be run *on-chain*, they
have to compile down to Plutus Core. This is why, by default,
datums, redeemers and contexts are all represented by the generic
`Data` type: it facilitates this compilation. We have however been
working with our *typed* validators. To permit them to be compiled
down into Plutus Code, we have to

### 1. Make our datum and redeemers instances of IsData

Being an instance of the `IsData` typeclass permits the program to
compile our datum and redeemer to Plutus Core. In production code,
`makeIsDataIndexed` is the preferred method, but for now we use
`unstableMakeIsData`:

```haskell
data MyDatum = MyDatum {...}

data MyRedeemer = MyRedeemer {...}

unstableMakeIsData ''MyDatum 
unstableMakeIsData ''MyRedeemer
```

The `''` in `''MyDatum` is Template Haskell's
way of turning the type `MyDatum` into a `Name` referring to the
`MyDatum` type identifier. It's boilerplate and does not need to
be understood, aside from the fact its presence is required.

### 2. Define a new validator data type

```haskell
data MyValidator
```

This data type is left intentionally empty, as its utility is
found in the next step.

### 3. Make the validator type an instance of ValidatorTypes

```haskell
data MyValidator 

instance ValidatorTypes MyValidator where 
  type instance RedeemerType MyValidator = MyRedeemer 
  type instance DatumType MyValidator = MyDatum
```

This is the mechanism by which we tell Plutus what our redeemer
and datum types *are*, allowing the compiler to check we are using
the write ones.

### 4. Construct a TypedValidator

`TypedValidator`s are ones that may actually be ran *on-chain*. We
have to utilise Template Haskell once again, in order to produce a
`TypedValidator`, but it is, once again, boilerplate:

```haskell
myValidator :: MyDatum -> MyRedeemer -> ScriptContext -> Bool 
myValidator d r ctx = ...

data MyValidator 

instance ValidatorTypes MyValidator where 
  type instance RedeemerType MyValidator = MyRedeemer 
  type instance DatumType MyValidator = MyDatum

myTypedValidator :: TypedValidator MyValidator
myTypedValidator = 
  mkTypedValidator @MyValidator
    $$(PlutusTx.compile [|| myValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
  where 
    wrap = wrapValidator @MyDatum @MyRedeemer
```

`mkTypedValidator` and `wrapValidator` are defined in
`Ledger.Typed.Scripts`.

## Conclusion

With this, we have a piece of validation logic, which may be ran
*on-chain*, whilst still granting us the power of Haskell's type
system.

## Examples of validators

### Validator which always succeeds

```haskell
myValidator _d _r _ctx = True
```

### Validator which always fails

```haskell
myValidator _d _r _ctx = False
```

### Validator which only passes if Datum is 42

```haskell
myDatum = myDatum { datVal :: Integer }

myValidator d _ _ = 
  traceIfFalse "datum value is not 42" $ 
    datVal d == 42
```
