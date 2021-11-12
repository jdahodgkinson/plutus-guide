# Constraints and Lookups

## Lookup Table

| TxConstraints i o              | ScriptLookups a         |
|--------------------------------|-------------------------|
| `mustPayToTheScript`           | `typedValidatorLookups` |
| `mustPayToPubKey`              | `unspentOutputs`        |
| `mustMintCurrency`             | `mintingPolicy`         |
| `mustMintCurrencyWithRedeemer` | `otherScript`           |
| `mustMintValue`                | `otherData`             |
| `mustMintValueWithRedeemer`    | `ownPubKeyHash`         |
| `mustSpendAtLeast`             | `pubKey`                |
| `mustSpendPubKeyOutput`        |
| `mustSpendScriptOutput`        |
| `mustValidateIn`               |
| `mustBeSignedBy`               |
| `mustIncludeDatum`             |
| `mustPayToOtherScript`         |
| `mustHashDatum`                |
