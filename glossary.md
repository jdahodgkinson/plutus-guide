Glossary
========

### AssetClass

AssetClass is a newtype wrapper around a tuple of `CurrencySymbol`
and `TokenName` i.e. 

```haskell
newtype AssetClass = AssetClass 
  { unAssetClass :: (CurrencySymbol, TokenName) 
  }
```

### CurrencySymbol
CurrencySymbol has type `BuiltinByteString` and refers to the hash
of the `MintingPolicy`, which forged the token in question. ada's
CurrencySymbol is the empty string `""`, as it cannot be forged
and therefore has no `MintingPolicy`.

### datum 
A piece of data held by a UTxO and used to carry script state 
information, such as its owner.

### redeemer
Arbitrary information included in a transaction, in 
order to provide an input to a script.

### TokenName
TokenName has type `BuiltinByteString` and is a human-readable
name of a token. ada's TokenName is the empty string `""`.

### UTxO
An Unspent Transaction Output.

### Value
In Plutus, Value is a newtype wrapper around a `Map` of 
`CurrencySymbol` to another `Map` of `TokenName` to `Integer` i.e. 

```haskell 
newtype Value = Value 
  { getValue :: Map CurrencySymbol (Map TokenName Integer) 
  }
```

It is used to represent the type and number of tokens in a given
transaction.
