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
