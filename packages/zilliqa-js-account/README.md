# @zilliqa-js/account
> Classes for managing accounts and account-related actions.

# Interfaces

```typescript
export interface TxReceipt {
  success: boolean;
  cumulative_gas: number;
}

interface TxParams {
  version: number;
  toAddr: string;
  amount: BN;
  gasPrice: BN;
  gasLimit: Long;

  code?: string;
  data?: string;
  receipt?: TxReceipt;
  nonce?: number;
  pubKey?: string;
  signature?: string;
}
```

# Classes

## `Account`

Class for managing an account (i.e., a private/public keypair).

### `Account(privateKey: string, nonce?: number): Account`

**Parameters**

- `privateKey`: `string` - hex-encoded private key
- `nonce`: `number` (optional) - the current nonce

**Returns**

- `Account` - an `Account` instance.

## Members

### `privateKey: string`

### `publicKey: string`

### `address: string`

### `nonce: number`

## Static Methods

### `static fromFile(file: string, passphrase: string): Promise<Account>`

Generates an account from any JSON-encoded string that complies with the [Web3 Secret Storage definition](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition).

**Parameters**

- `file`: `string` - JSON-encoded string containing the keystore file.
- `passphrase`: `string` - passphrase used to encrypt the file.

**Returns**

- `Promise<Account>` - an `Account` instance, initialised with the details
  provided by the keystore file.

## Instance Methods

### `toFile(passphrase: string, kdf: 'pbkdf2' |'scrypt' = 'scrypt'): Promise<Account>`

Encrypts and JSON-encodes the account. Complies with the [Web3 Secret Storage definition](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition).

**Parameters**

- `passphrase`: `string` - passphrase used to encrypt the file.
- `kdf`: `'pbkdf2' | 'scrypt'` - the key derivation function to use for
  encryption.

**Returns**

- `Promise<string>` - the JSON-encoded string of the keystore file.

### `signTransaction(bytes: Buffer): string`

Signs arbitrary bytes (most often transactions) using a Schnorr signing
scheme.

**Parameters**

- `bytes`: `Buffer` - a `Buffer` of the `protobuf` encoded transaction bytes.

**Returns**

- `string` - hex-encoded signature over the bytes, using the instance private
  key.

## `Wallet`

Class for managing multiple accounts.

### `Wallet(provider: Provider, accounts?: Account[] = []): Wallet`

**Parameters**

- `provider`: `Provider` - a Provider instance (see
  `@zilliqa-js/core`). Required for signing.
- `accounts`: `Account[]` (optional) - an array of `Account` instances to
  pre-populate the wallet with.

**Returns**

- `Wallet`

## Members

### `accounts: { [address: string]: Account }`

An object consisting of `address: Account` KV pairs. By default, an empty
object.

### `defaultAccount: Account`

The default account used for signing transactions. By default, `undefined`. It
is set to the `0`-indexed account when a `Wallet` instance is constructed.

## Instance methods

### `create(): void`

Creates a new keypair with a randomly-generated private key. The new account is accessible by address. This method mutates the `Wallet` instance.

**Parameters**

None

**Returns**

- `string` - address of the new account.

### `addByPrivateKey(privateKey: string): string`

Adds an `Account` to the `Wallet`.

**Parameters**

- `privateKey`: `string` - hex-encoded private key.

**Returns**

- `string` - the corresponing address, computer from the private key.

### `addByKeystore(keystore: string, passphrase: string): Promise<string>`

 Adds an account by keystore. This method is asynchronous and returns a `Promise<string>`, in order not to block on the underlying decryption operation.

**Parameters**

- `keystore`: `string` - JSON-encoded keystore file.
- `passphrase`: `string` - the passphrase used to encode the keystore file.

**Returns**

- `Promise<string>` - the corresponding address.

### `addByMnemonic(phrase: string, index: number = 0): string`

Adds an `Account` by use of a mnemonic as specified in BIP-32 and BIP-39

**Parameters**

- `phrase`: `string` - the 12-word mnemonic to use.
- `index`: `number` (Optional) - the index of the child key.

**Returns**

- `string` - the corresponding address.

### `export(address: string, passphrase, string, kdf: 'pbkdf2' | 'scrypt'): Promise<string>`

- Exports an `Account` to a keystore file, encrypted with a passphrase.

**Parameters**

- `address`: `string` - the address of the selected account.
- `passphrase`: `string` - the passphrase to encrypt the `Account` with.
- `kdf`: `'pbkdf2' | 'scrypt'` - key derivation function.

**Returns**

- `Promise<string>` - the JSON-encoded keystore file.

### `remove(address: string): boolean`

- Exports an `Account` to a keystore file, encrypted with a passphrase.

**Parameters**

- `address`: `string` - the address of the account to remove.

**Returns**

- `boolean` - whether the `Account` was successfully removed.

### `setDefault(address: string): void`

Sets the default account to sign with.

**Parameters**

- `address`: `string` - the address of the account to set as default.

**Returns**

- `void`

### `sign(transaction: Transaction): Promise<Transaction>`

Sign a `Transaction` with the default `Account`. This method is asynchronous
as it will attempt to obtain the `nonce` from the `Provider`.

**Parameters**

- `transaction`: `Transaction` - a `Transaction` instance.

**Returns**

- `Promise<Transaction>` - a signed transaction.

### `signWith(transaction: Transaction, address: string): Promise<Transaction>`

Sign a `Transaction` with the chosen `Account`. This method is asynchronous
as it will attempt to obtain the `nonce` from the `Provider`.

**Parameters**

- `transaction`: `Transaction` - a `Transaction` instance.
- `address`: `string` - the address of the `Account` to be used for signing.

**Returns**

- `Promise<Transaction>` - a signed transaction.

## `Transaction`

A class that represents a single `Transaction` on the Zilliqa network. It is a functor. Its purpose is to encode the possible states a Transaction can be in:  Confirmed, Rejected, Pending, or Initialised (i.e., not broadcasted).

## Members

### `bytes: Buffer`

A getter `protobuf` that returns a `Buffer` of `protobuf`-encoded bytes. This
is a convenience member that allows a `Transaction` to be signed easily.

### `senderAddress: string`

A getter than computes the address of the `Transaction` sender. If there is no
sender public key set, returns `0x00000000000000000000000000000000000000000000`.

### `txParams: TxParams`

A getter that returns the current `TxParams`.

## Static Methods

### `static confirm(params: TxParams, provider: Provider): Transaction`

Instantiates a `Transaction` in `Confirmed` state.

**Parameters**

- `params`: `TxParams` - core fields to initialise the `Transaction` with.
- `provider`: `Provider` - a `Provider` instance.

**Returns**

- `Transaction` - the newly-Instantiated `Transaction`.

**Example**

```typescript
import { HTTPProvider } from '@zilliqa-js/core';
import { Transaction } from '@zilliqa-js/account';

const txParams = {
  version: 0,
  toAddr: '20_byte_hex_string',
  amount: new BN(8),
  gasPrice: new BN(100),
  gasLimit: Long.fromNumber(888)
};
const tx = Transaction.confirm(txParams, new HTTPProvider('http://my-api.com'));

expect(tx.isConfirmed()).toBeTruthy();
```

### `static reject(params: TxParams, provider: Provider): Transaction`

Instantiates a `Transaction` in `Rejected` state.

**Parameters**

- `params`: `TxParams` - core fields to initialise the `Transaction` with.
- `provider`: `Provider` - a `Provider` instance.

**Returns**

- `Transaction` - the newly-Instantiated `Transaction`.

**Example**

```typescript
import { HTTPProvider } from '@zilliqa-js/core';
import { Transaction } from '@zilliqa-js/account';

const txParams = {
  version: 0,
  toAddr: '20_byte_hex_string',
  amount: new BN(8),
  gasPrice: new BN(100),
  gasLimit: Long.fromNumber(888),
};
const tx = Transaction.reject(txParams, new HTTPProvider('http://my-api.com'));
expect(tx.isRejected()).toBeTruthy();
```

## Instance Methods

### `confirm(txHash: string, maxAttempts: number = 33, interval: number = 1000): Promise<Transaction>`

Checks whether the `Transaction` is confirmed on the blockchain, by verifying
the its `receipt` status (`boolean`). This method uses an exponential backoff
to poll the lookup node. By default, the number of attempts made is 33, with
a starting interval of 1000ms.

**Parameters**

- `txHash`: `string` - the transaction hash to use for polling.
- `maxAttempts`: `number = 33` (Optional) - the maximum number of attempts
  before setting status as `Rejected`.
- `interval`: `number = 1000` (Optional) - the initial interval. This grows
  exponentially between attempts.

**Returns**

- `Promise<Transaction>` - `Transaction` with its status confirmed onchain.

**Example**

```typescript
import { HTTPProvider } from '@zilliqa-js/core';
import { Transaction } from '@zilliqa-js/account';

// hash can be obtained from CreateTransaction
const my_hash = 'some_known_tx_hash';
conts tx = new Transaction(params, new HTTPProvider('http://my-api.com'));

tx.confirm(some_hash)
  .map((tx) => // do something)
  .catch((err) => // handle the error);
```

### `map(txHash): Transaction`

Maps over the transaction, taking a callback that accepts `TxParams`. The user
may freely mutate the Transaction, and will receive the newly-mutated
transaction. The object returned is merged into the target `Transaction`.

**Parameters**

- `fn`: `(prev: TxParams) => TxParams)` - the transaction hash to use for polling.
  exponentially between attempts.

**Returns**

- `Transaction`.

**Example**

```typescript
import { HTTPProvider } from '@zilliqa-js/core';
import { Transaction } from '@zilliqa-js/account';

// hash can be obtained from CreateTransaction
const my_hash = 'some_known_tx_hash';
let tx = new Transaction(params, new HTTPProvider('http://my-api.com'));

async () => {
  try {
    tx = await tx.confirm(some_hash);
    if (tx.isConfirmed()) {
      .map((tx) => {
        // do something, but must always return `TxParams`.
        // generally, you should avoid performing side effects in `map`.
        return tx
      });
    }
  } catch (err) {
    // handle this error somehow
  }
}();
```

# Functions

### `encodeTransactionProto(tx: TxParams): Buffer`

Encodes a transaction with `protobuf` and returns its bytes as a Buffer. Used
for providing a payload to `signTransaction`.

**Parameters**

- `tx`: `TxParams` - plain object containing core transaction fields that must
be used when generating a signature.

**Returns**

- `Buffer` - the bytes of the `protobuf`-serialised transaction fields.

