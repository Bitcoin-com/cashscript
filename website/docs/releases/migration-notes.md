---
title: Migration Notes
---

## v0.5 to v0.6
#### cashc compiler
The exports for library usage of `cashc` have been updated. All utility-type exports have been moved to the `@cashscript/utils` package, but they are still accessible from the `utils` export from `cashc`. Note that the recommended use of `cashc` is still the CLI, not the NPM package.

In v0.5 you could encode a string like this:
```js
const { Data } = require('cashc');

const encodedString = Data.encodeString('Hello World');
```

While for v0.6 you'd need to use the `utils` export or `@cashscript/utils`:
```js
const { utils } = require('cashc');
const { encodeString } = require('@cashscript/utils');

const encodedString = utils.encodeString('Hello World');
const encodedString = encodeString('Hello World);
```

---

Compilation functions used to be exported as part of the `CashCompiler` object, but are now exported as standalone functions.

In v0.5 compilation looked like this:
```js
const { CashCompiler } = require('cashc');

const Mecenas = CashCompiler.compileFile(path.join(__dirname, 'mecenas.cash'));
```

In v0.6, this needs to be changed to this:
```js
const { compileFile } = require('cashc');

const Mecenas = compileFile(path.join(__dirname, 'mecenas.cash'));
```

#### CashScript SDK
The CashScript SDK no longer depends on `cashc` and no longer exports the `CashCompiler` object. This reflects the recommended usage where the CLI is used for compilation and the artifact JSON is saved. Then this artifact JSON can be imported into the CashScript SDK. If you prefer to compile your contracts from code, you need to add `cashc` as a dependency and use its compilation functionality.

## v0.4 to v0.5
#### CashScript SDK
The contract instantiation flow has been refactored to enable compatibility with more BCH libraries and simplify the different classes involved.

---

In v0.4 a contract could be compiled or imported using `Contract.compile()` or `Contract.import()`, which returned a Contract object. On that Contract object `contract.new(...args)` could be called, which returned an Instance object. In the v0.5 release, the Contract and Instance objects have been merged and simplified, while the compilation has been extracted into its own class.

In v0.4, contract instantiation looked like this:
```js
const { Contract } = require('cashscript');

const Mecenas = Contract.compile(path.join(__dirname, 'mecenas.cash'), 'testnet');
const contract = Mecenas.new(alicePkh, bobPkh, 10000);
```

In v0.5, this needs to be changed to look like this:
```js
const { CashCompiler, ElectrumNetworkProvider, Contract } = require('cashscript');

const Mecenas = CashCompiler.compileFile(path.join(__dirname, 'mecenas.cash'));
const provider = new ElectrumNetworkProvider('testnet');
const contract = new Contract(Mecenas, [alicePkh, bobPkh, 10000], provider);
```

---

* Transaction object's `.send()` function now returns either a libauth Transaction or raw hex string rather than a BITBOX Transaction. If it is necessary, the raw hex string can be imported into libraries such as BITBOX to achieve similar functionality as before.

* In v0.4.1, `Sig` was deprecated in favour of `SignatureTemplate`. In v0.5.0, the deprecated class has been removed. All occurrences of `Sig` should be replaced with `SignatureTemplate`.

See the [release notes](/docs/releases/release-notes#v050) for an overview of other new changes.

## v0.3 to v0.4
#### cashc compiler
In v0.3, casting an `int` type to a `bytes` would perform an `NUM2BIN` operation, padding the value to 8 bytes. This made `bytes(10)` equivalent to `bytes8(10)`. From v0.4.0 onwards, casting to an *unbounded* `bytes` type is only a semantic cast, indicating that the `int` value should be treated as a `bytes` value.

* If you need the old behaviour, you should change all occurrences of `bytes(x)` to `bytes8(x)`.

#### CashScript SDK
The entire `Transaction` flow has been refactored to a more fluent chained TransactionBuilder API.

* All occurrences of `.send(to, amount)` should be replaced with `.to(to, amount).send()`.
* All occurrences of `.send(outputs)` should be replaced with `.to(outputs).send()`.
  * Alternatively, the list of `outputs` can be split up between several `.to()` calls.
  * If any of the outputs contain `opReturn` outputs, these should be added separately using `.withOpReturn(chunks)`
* The same transformations are applicable to all `.meep()` calls.
* The `meep()` function previously logged the meep command automatically, but now it returns the command as a string, so you should `console.log()` the command separately.
* All transaction options previously included in the `TxOptions` object should now be provided using chained functions.
  * The `time` option should be provided using the `.withTime(time)` function.
  * The `age` option should be provided using the `.withAge(age)` function.
  * The `fee` option should be provided using the `.withHardcodedFee(fee)` function.
  * The `minChange` option should be provided using the `.withMinChange(minChange)` function.

In v0.2.2, `Contract.fromCashFile()` and `Contract.fromArtifact()` were deprecated in favour of `Contract.compile()` and `Contract.import()`. In v0.4.0, the deprecated functions have been removed.

* All occurrences of `Contract.fromCashFile()` should be replaced with `Contract.compile()`.
* All occurrences of `Contract.fromArtifact()` should be replaced with `Contract.import()`.

See the [release notes](/docs/releases/release-notes#v040) for an overview of other new changes.
