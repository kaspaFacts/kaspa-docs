# Keys and Addresses

Alice wants to pay Bob, but the thousands of Kaspa full nodes who will verify her transaction don't know who Alice or Bob are--and we want to keep it that way to protect their privacy. Alice needs to communicate that Bob should receive some of her kaspa without tying any aspect of that transaction to Bob's real-world identity or to other Kaspa payments that Bob receives. The method Alice uses must ensure that only Bob can further spend the kaspa he receives.

The original Kaspa design describes a very simple scheme for achieving those goals, shown in the following illustration.

A receiver like Bob accepts kaspa to a public key in a transaction that is signed by the spender (like Alice). The kaspa that Alice is spending had been previously received to one of her public keys, and she uses the corresponding private key to generate her signature. Full nodes can verify that Alice's signature commits to the output of a hash function that itself commits to Bob's public key and other transaction details.

We'll examine public keys, private keys, signatures, and hash functions in this chapter, and then use all of them together to describe the addresses used by modern Kaspa software.

---

## Public Key Cryptography

Public key cryptography was invented in the 1970s and is a mathematical foundation for modern computer and information security.

Since the invention of public key cryptography, several suitable mathematical functions, such as prime number exponentiation and elliptic curve multiplication, have been discovered. These mathematical functions are easy to calculate in one direction and infeasible to calculate in the opposite direction using the computers and algorithms available today. Based on these mathematical functions, cryptography enables the creation of unforgeable digital signatures. Kaspa uses elliptic curve addition and multiplication as the basis for its cryptography.

In Kaspa, we can use public key cryptography to create a key pair that controls access to kaspa. The key pair consists of a private key and a public key derived from the private key. The public key is used to receive funds, and the private key is used to sign transactions to spend the funds.

There is a mathematical relationship between the public and the private key that allows the private key to be used to generate signatures on messages. These signatures can be validated against the public key without revealing the private key.

> **Note:** In some wallet implementations, the private and public keys are stored together as a _key pair_ for convenience. However, the public key can be calculated from the private key, so storing only the private key is also possible.

A Kaspa wallet contains a collection of key pairs, each consisting of a private key and a public key. The private key (_k_) is a number, usually derived from a number picked at random. From the private key, we use elliptic curve multiplication, a one-way cryptographic function, to generate a public key (_K_).

### Why Use Asymmetric Cryptography (Public/Private Keys)?

Why is asymmetric cryptography used in Kaspa? It's not used to "encrypt" (make secret) the transactions. Rather, a useful property of asymmetric cryptography is the ability to generate _digital signatures_. A private key can be applied to a transaction to produce a numerical signature. This signature can only be produced by someone with knowledge of the private key. However, anyone with access to the public key and the transaction can use them to _verify_ the signature. This useful property of asymmetric cryptography makes it possible for anyone to verify every signature on every transaction, while ensuring that only the owners of private keys can produce valid signatures.

---

## Private Keys and Elliptic Curve Cryptography

In the previous section, we introduced the concept of public key cryptography. Now we'll dive deeper into the mathematical foundations that make Kaspa's security possible: private keys, elliptic curve cryptography, and how public keys are derived from private keys.

### What is a Private Key?

A private key is essentially a randomly generated number—a 256-bit integer chosen from a very large range. In Kaspa, private keys are typically represented as 32-byte arrays. The security of the entire system depends on the randomness of this number: if someone can guess or derive your private key, they can spend your funds.

Private keys are generated using cryptographically secure random number generators. In practice, users rarely work with raw private keys directly. Instead, wallets generate them from mnemonic seed phrases using BIP32 hierarchical deterministic (HD) key derivation, which we'll cover later in this chapter.

### Elliptic Curve Cryptography

Kaspa uses the secp256k1 elliptic curve, the same curve used by Bitcoin. This curve was chosen for its well-studied security properties and efficient implementation. The curve is defined over a finite field, meaning all calculations happen within a mathematical structure with a fixed number of elements.

The secp256k1 curve has a special point called the "generator point" (denoted as G). This point serves as the foundation for public key derivation. The curve's order (the number of points on the curve) is approximately 2^256, which provides an enormous security margin.

### Public Key Derivation

The relationship between private keys and public keys is defined by elliptic curve multiplication. Given a private key k (a number) and the generator point G, the public key K is calculated as:

```
K = k × G
```

This operation is elliptic curve point multiplication, not ordinary multiplication. The critical property is that while calculating K from k is straightforward, deriving k from K is computationally infeasible—this is the "elliptic curve discrete logarithm problem" that provides the security foundation.

### Schnorr vs ECDSA Signatures

Kaspa supports two signature schemes: Schnorr (the default and recommended) and ECDSA (for compatibility).

**Schnorr Signatures** are the modern standard in Kaspa. They offer several advantages:

- **Simpler security proofs**: Schnorr signatures have cleaner mathematical properties
- **Better efficiency**: Smaller signature sizes (64 bytes vs 65-72 bytes for ECDSA)
- **Linearity**: Multiple signatures can be combined in ways that enable advanced features like multisig
- **x-only public keys**: Schnorr uses 32-byte x-only public keys, saving space

**ECDSA Signatures** are supported for compatibility with older systems and certain use cases. ECDSA uses full 33-byte public keys (including the y-coordinate parity bit) and produces slightly larger signatures.

When creating addresses, wallets can choose between Schnorr (default, `Version::PubKey`) or ECDSA (`Version::PubKeyECDSA`) based on the public key format.

### Key Security Considerations

- **Never share your private key**: Anyone with your private key can spend your funds
- **Use secure random generation**: Private keys must be generated with cryptographically secure randomness
- **Back up your mnemonic**: Since private keys are derived from mnemonics in HD wallets, backing up your mnemonic phrase is equivalent to backing up all your keys
- **Beware of phishing attacks**: Never enter your private key or mnemonic into untrusted software or websites

With this foundation in private keys and elliptic curve cryptography, we can now understand how Kaspa addresses encode these public keys for use in transactions.

---

## Bech32 Address Encoding

Kaspa addresses use bech32 encoding from the start of the protocol. Unlike Bitcoin's transition from base58check to bech32 with segwit, Kaspa was designed with bech32 as its native address format. This choice provides several advantages for users and developers.

### Address Structure

A Kaspa address consists of three parts:

- **Prefix**: Identifies the network (mainnet, testnet, simnet, or devnet)
- **Version**: Identifies the script type (PubKey, PubKeyECDSA, or ScriptHash)
- **Payload**: The underlying data (32-byte public key for Schnorr, 33-byte public key for ECDSA, or 32-byte script hash)

The bech32 encoding combines these three parts into a human-readable string with a checksum for error detection.

### Network Prefixes

Kaspa uses different prefixes to distinguish between networks:

| Prefix | Network |
|--------|----------|
| kaspa | Mainnet |
| kaspatest | Testnet |
| kaspasim | Simnet (simulation) |
| kaspadev | Devnet (development) |

The prefix is followed by a separator "1" and then the encoded version and payload.

### Address Versions

Kaspa supports three address versions, each corresponding to a different script type:

| Version | Byte Value | Payload Length | Description |
|---------|------------|----------------|-------------|
| PubKey | 0 | 32 bytes | Schnorr public key (default) |
| PubKeyECDSA | 1 | 33 bytes | ECDSA public key (legacy compatibility) |
| ScriptHash | 8 | 32 bytes | Blake2b hash of a script (P2SH) |

The version byte is encoded as the first character of the data part after the separator.

### Encoding Process

The bech32 encoding process works as follows:

1. **Combine version and payload**: The version byte and payload bytes are concatenated.
2. **Convert to 5-bit groups**: The 8-bit data is converted to 5-bit groups using bit manipulation.
3. **Calculate checksum**: A BCH (Bose-Chaudhuri-Hocquenghem) code checksum is calculated using the `polymod` function.
4. **Map to charset**: Each 5-bit value is mapped to a character from the bech32 alphabet: `qpzry9x8gf2tvdw0s3jn54khce6mua7l`
5. **Combine with prefix**: The prefix, separator, encoded data, and checksum are concatenated to form the final address.

Kaspa uses standard bech32 encoding (BIP 173), not bech32m (BIP 350). The checksum constant is `0x2bc830a3` for bech32, which provides strong error detection for the defined address lengths.

### Decoding Process

Decoding reverses the encoding process:

1. **Separate parts**: The address is split into prefix, separator, data part, and checksum.
2. **Validate checksum**: The checksum is recalculated and compared to detect errors.
3. **Convert to 8-bit**: The 5-bit data is converted back to 8-bit bytes.
4. **Extract version and payload**: The first byte is the version, and the remaining bytes are the payload.

### Advantages of Bech32

Kaspa's use of bech32 provides several benefits:

- **Case-insensitive**: Bech32 uses only lowercase letters, making it easier to read and transcribe. Uppercase can be used for QR codes to save space.
- **Error detection**: The BCH checksum can detect up to 4 character substitutions and many transposition errors.
- **Future-proof**: The version byte allows for new address types to be added without changing the encoding format.
- **QR code efficiency**: Lowercase-only encoding allows for more compact QR codes.

### Example Address

Sample Kaspa mainnet address:

```
kaspa:qpq64dfdztjjzte7xmqt9ehe4n456xg4v30ekqlveut6u3wfhzyasq9whp8ak
```

This is a standard PubKey (Schnorr) address on the Kaspa mainnet. The address encodes a 32-byte x-only public key with version byte 0, followed by a bech32 checksum for error detection.

---

## Address Versions

Kaspa addresses use a version byte to identify the type of script and public key format. Unlike Bitcoin's witness versions, Kaspa's version byte directly indicates the script class and key encoding. There are three standard address versions in Kaspa.

### PubKey (Version 0)

PubKey addresses are the default and most common address type in Kaspa. They use Schnorr signatures with 32-byte x-only public keys. The script is simple and efficient:

```
<OP_DATA_32> <32-byte public key> <OP_CHECKSIG>
```

This script checks that a valid Schnorr signature is provided for the embedded public key. The total script length is 34 bytes.

PubKey addresses are recommended for most use cases because:
- Schnorr signatures provide better security and simplicity than ECDSA
- 32-byte keys are more compact than 33-byte ECDSA keys
- The script is minimal and easy to verify

### PubKeyECDSA (Version 1)

PubKeyECDSA addresses provide compatibility with ECDSA signatures, using 33-byte compressed public keys. The script format is:

```
<OP_DATA_33> <33-byte public key> <OP_CHECKSIGECDSA>
```

This version exists for interoperability with systems that require ECDSA, such as certain hardware wallets or legacy integrations. The total script length is 35 bytes.

While ECDSA is widely supported in cryptographic libraries, Schnorr is preferred for new implementations due to its simpler security proofs and better multi-signature support.

### ScriptHash (Version 8)

ScriptHash addresses implement Pay-to-Script-Hash (P2SH), allowing payments to complex scripts without revealing the spending conditions upfront. The script format is:

```
<OP_BLAKE2B> <OP_DATA_32> <32-byte script hash> <OP_EQUAL>
```

The 32-byte hash is computed using Blake2b on the redeem script. When spending, the spender provides the redeem script along with signatures, and the script engine verifies that the hash matches.

P2SH enables advanced features like:
- Multi-signature wallets requiring M-of-N signatures
- Time-locked payments
- Complex spending conditions
- Anyone-can-spend scripts for testing

The version byte 8 was chosen to distinguish ScriptHash from the key-based versions (0 and 1).

### Version Selection

Wallets typically default to PubKey addresses for new receive addresses. The choice depends on:

- **Security requirements**: Schnorr (PubKey) is preferred for its simplicity
- **Compatibility**: Use PubKeyECDSA only if ECDSA is required
- **Complexity**: Use ScriptHash for multi-sig or custom spending conditions

---

## Pay-to-Public-Key (P2PK)

Pay-to-Public-Key (P2PK) is the simplest and most common way to receive payments in Kaspa. Unlike Bitcoin, which historically used Pay-to-Public-Key-Hash (P2PKH) as its primary address type, Kaspa uses P2PK directly. This design choice simplifies the script and reduces computational overhead while maintaining security.

### How P2PK Works

In a P2PK transaction, the output script contains the recipient's public key directly. The script is very simple:

```
<OP_DATA_32> <32-byte public key> <OP_CHECKSIG>
```

This script does exactly what it says: it pushes a 32-byte public key onto the stack, then executes `OP_CHECKSIG`, which verifies that a valid Schnorr signature is provided for that public key.

The total script length is 34 bytes: 1 byte for `OP_DATA_32`, 32 bytes for the public key, and 1 byte for `OP_CHECKSIG`.

### Why Kaspa Uses P2PK Instead of P2PKH

Bitcoin's original design used P2PKH (Pay-to-Public-Key-Hash), where the address contains a hash of the public key rather than the public key itself. This was done to save space (20-byte hash vs 33-65 byte public key) and provide some protection against quantum computers (though this protection is theoretical).

Kaspa chose P2PK for several reasons:

- **Simpler scripts**: P2PK scripts are shorter and simpler to validate
- **No hash preimage requirement**: Spending doesn't require revealing the public key (it's already in the script)
- **Better efficiency**: Modern hardware handles 32-byte public keys efficiently
- **Security**: The security of elliptic curve cryptography is well-established; the theoretical quantum threat is not practical

### P2PK Transaction Flow

When Alice wants to pay Bob using P2PK:

1. **Bob provides his address**: Bob's wallet generates a public key from his private key and encodes it as a Kaspa address (using bech32 encoding with version 0 for PubKey)
2. **Alice creates a transaction**: Alice's wallet constructs a transaction with an output script containing Bob's public key
3. **Alice signs the transaction**: Alice uses her private key to create a Schnorr signature for each input she's spending
4. **Network validation**: When the transaction is broadcast, full nodes verify that Alice's signatures are valid for the inputs she's spending
5. **Bob can spend**: Later, when Bob wants to spend the output, he provides his signature along with the transaction. The script engine executes `OP_CHECKSIG` which verifies that Bob's signature matches the public key in the script

The signature script for spending a P2PK output is simply the signature itself (64 bytes for Schnorr) followed by the sighash type byte.

### P2PK with ECDSA

For compatibility, Kaspa also supports P2PK with ECDSA signatures. The script structure is similar but uses a 33-byte public key and `OP_CHECKSIGECDSA`:

```
<OP_DATA_33> <33-byte public key> <OP_CHECKSIGECDSA>
```

This is used when addresses are created with `Version::PubKeyECDSA`.

P2PK is the foundation of Kaspa's payment system. While more complex script types like P2SH enable advanced features, P2PK remains the most common and efficient way to receive payments.

---

## The Kaspa Script System

Kaspa uses a stack-based script system similar to Bitcoin's but with important differences. Scripts define the conditions under which transaction outputs can be spent. When a transaction is validated, the script engine executes the script to determine if the spending is authorized.

### Script Structure

Each transaction output contains a `ScriptPublicKey` that specifies the spending conditions. Each transaction input provides a `signature_script` that satisfies those conditions. The script engine executes both scripts in sequence to validate the transaction.

### Script Classes

Kaspa recognizes four standard script classes:

- **NonStandard**: Scripts that don't match any recognized pattern
- **PubKey**: Direct pay-to-public-key with Schnorr signatures
- **PubKeyECDSA**: Pay-to-public-key with ECDSA signatures
- **ScriptHash**: Pay-to-script-hash for complex spending conditions

### Script Execution

The script engine uses two stacks:
- **Data stack (dstack)**: Holds values being manipulated
- **Alt stack (astack)**: Used by certain opcodes for temporary storage

Scripts are executed opcode by opcode, with each opcode performing operations on the stacks. The script is considered valid if it completes without error and leaves a `true` value on top of the data stack.

For P2SH scripts, execution happens in two phases:
1. The signature script and script public key are executed normally
2. If successful, the stack is restored and the redeem script (from the signature script) is executed

### Standard Script Sizes

Kaspa defines a maximum standard script size to prevent denial-of-service attacks. The standard script public key sizes are:
- P2PK: 34 bytes
- P2PK-ECDSA: 35 bytes
- P2SH: 35 bytes

Scripts exceeding the standard size may still be valid but will incur additional storage mass.

### Opcodes

Kaspa supports a variety of opcodes for data manipulation, control flow, and cryptographic operations. These include:
- Data push opcodes (OpData1 through OpData32, OpPushData1/2/4)
- Cryptographic opcodes (OpCheckSig, OpCheckSigECDSA, OpBlake2b)
- Control flow opcodes (OpIf, OpElse, OpEndIf, OpVerify)
- Stack manipulation opcodes (OpDrop, OpDup, OpPick, etc.)

The script engine enforces resource limits through script units, which account for the computational cost of script execution.

---

## Pay-to-Script-Hash (P2SH)

Pay-to-Script-Hash (P2SH) is a powerful script type that allows complex spending conditions to be encoded as a hash. Instead of requiring the spender to know the full spending conditions, P2SH only requires them to provide a script that hashes to a specific value. This enables advanced features like multisig wallets, time-locked transactions, and other custom spending logic.

### How P2SH Works

In a P2SH transaction, the output script contains only a 32-byte hash of the redeem script (the actual spending conditions). The spender provides the redeem script along with the signature when spending the output. The script engine verifies that:

1. The provided redeem script hashes to the value in the output script
2. The redeem script executes successfully with the provided signatures

### P2SH Script Format

The P2SH script format is simple and efficient:

```
<OP_BLAKE2B> <OP_DATA_32> <32-byte script hash> <OP_EQUAL>
```

This script checks that the Blake2b hash of the provided redeem script matches the hash embedded in the output. The total script length is 35 bytes.

### Creating P2SH Addresses

To create a P2SH address, you:

1. Construct the redeem script with your desired spending conditions (e.g., a 2-of-3 multisig script)
2. Compute the Blake2b hash of the redeem script
3. Create a ScriptHash address with version 8 using this hash as the payload

### Spending from P2SH

To spend from a P2SH output, the signature script must contain:

1. The signatures required by the redeem script
2. The complete redeem script itself

### Use Cases

P2SH enables several important use cases:

- **Multisig wallets**: Require multiple signatures to spend funds (e.g., 2-of-3 for corporate accounts)
- **Time-locked transactions**: Funds that can only be spent after a certain time or block height
- **Complex spending conditions**: Custom logic for spending that goes beyond simple signature verification
- **Privacy**: The spending conditions are hidden until the transaction is spent

### Multisig Support

Kaspa supports multisig redeem scripts through functions that create scripts requiring M-of-N signatures, where M signatures out of N possible signers are required to spend.

The script engine enforces resource limits through script units to prevent abuse of complex scripts.

---

## Collision Resistance and Hash Function Security

When using hash-based addresses like Pay-to-Script-Hash (P2SH), a theoretical security concern is the possibility of a _collision attack_. A collision occurs when two different inputs produce the same hash output. If an attacker could find a collision, they could create a different script that hashes to the same address as yours, potentially allowing them to spend your funds.

### The Security of Blake2b

Kaspa uses Blake2b as its primary hash function for P2SH addresses and other protocol primitives. Blake2b is a modern cryptographic hash function that provides several security advantages over older hash functions:

- **256-bit output**: Blake2b produces a 256-bit hash, providing 128 bits of collision resistance. This means an attacker would need to perform approximately 2^128 hash operations to find a collision—a task that would take all current computing power billions of years.
- **Domain separation**: Kaspa uses domain-separated Blake2b hashers for different purposes. Each hasher is initialized with a unique domain separator (such as `b"TransactionID"` or `b"BlockHash"`), which prevents cross-protocol attacks where a collision in one context could be exploited in another.
- **Built-in key support**: Blake2b natively supports keyed hashing, which Kaspa uses for additional security in certain contexts.

### Comparison with Bitcoin's Approach

Bitcoin's original P2SH used a combination of SHA256 and RIPEMD160, resulting in a 160-bit hash with only 80 bits of collision resistance. While this was considered adequate at the time, advances in computing power and cryptographic research have made stronger hash functions preferable.

Kaspa's design benefits from being developed later, allowing it to use Blake2b from the start without requiring a transition like Bitcoin's move to bech32 addresses. The 256-bit hash output provides a much larger security margin against collision attacks.

### Practical Security Considerations

While collision attacks are theoretically possible, they are not considered a practical threat for most users:

- The computational cost of finding a Blake2b collision is astronomically high
- Domain separation means a collision in one context (e.g., transaction IDs) cannot be exploited in another (e.g., addresses)
- The script engine enforces resource limits that prevent abuse of complex hashing operations

For these reasons, P2SH addresses in Kaspa provide strong security against collision attacks, and users can confidently use them for complex spending conditions like multisig wallets and time-locked transactions.

---

## HD Wallets and Key Derivation

Hierarchical Deterministic (HD) wallets allow you to generate a tree of addresses from a single seed. This means you can back up your wallet with just one mnemonic phrase and recover all your addresses and funds. Kaspa uses BIP32 HD wallets with a custom derivation path optimized for the Kaspa network.

### Mnemonic Seed Phrases

Kaspa wallets typically start with a mnemonic phrase—a sequence of 12 or 24 words that encodes a random seed. This seed is used to generate a master extended private key (xprv), which is the root of your key tree. The mnemonic is your backup: anyone with access to it can recover your entire wallet.

From the mnemonic, the wallet derives a master key using BIP32 standards. Kaspa uses the `kprv` prefix for extended private keys and `kpub` for extended public keys.

### Kaspa Derivation Path

Kaspa uses a custom derivation path that differs from Bitcoin's BIP44 standard. There are two distinct paths depending on wallet type:

**For single-signature wallets:**

```
m/44'/111111'/<account>'/<address_type>
```

**For multisig wallets:**

```
m/45'/111111'/<account>'/<cosigner>/<address_type>
```

**Path components:**

- **Purpose**: `44` for single-signature, `45` for multisig
- **Coin Type**: `111111'` (Kaspa's registered BIP44 coin type)
- **Account**: Account index (0, 1, 2, ...) - hardened derivation
- **Cosigner**: Cosigner index for multisig (0, 1, 2, ...) - only present in multisig paths
- **Address Type**: `0` for receive addresses, `1` for change addresses

**Important:** The cosigner index is only included for multisig wallets and is required for multisig path derivation.

For a standard single-signature wallet, the first receive address uses: `m/44'/111111'/0'/0/0`

### Receive and Change Chains

Kaspa HD wallets maintain two separate address chains:

- **Receive Chain**: Addresses where you receive payments from others
- **Change Chain**: Addresses used to send change back to yourself when spending

Each chain derives addresses independently using the same parent key. This separation improves privacy by making it harder to link transactions.

### Extended Keys

Extended keys contain both a key (public or private) and a chain code used for derivation. There are two types:

- **Extended Private Key **(xprv): Can derive child private keys and public keys. Must be kept secret.
- **Extended Public Key **(xpub): Can only derive child public keys. Can be shared safely to generate watch-only wallets.

### Legacy vs Modern Derivation

Kaspa supports two derivation schemes:

- **Gen0 **(Legacy): Uses the `'972` derivation path for compatibility with older wallets
- **Gen1 **(Standard): Uses the `'111111'` derivation path, the current standard

New wallets should use Gen1 derivation. The wallet framework automatically handles the correct derivation based on the account type.

### Address Discovery

HD wallets need to discover which addresses have been used to receive funds. The wallet framework scans the blockchain for transactions involving addresses derived from your xpub, incrementing the index until it finds a gap of unused addresses. This ensures you don't miss any funds sent to addresses you haven't explicitly tracked.

---

## Summary

In this chapter, you learned how Kaspa uses public key cryptography to enable secure, private transactions without requiring trust in third parties. We explored the fundamental building blocks that allow Alice to pay Bob without revealing their identities and while ensuring only Bob can spend the received funds.

We covered:

- **Public Key Cryptography**: The foundation of Kaspa's security, using secp256k1 elliptic curve cryptography with Schnorr signatures as the default and ECDSA for compatibility

- **Private Keys and Elliptic Curve Cryptography**: How private keys are generated as random 256-bit integers, and how public keys are derived using the formula K = k × G on the secp256k1 curve

- **Bech32 Address Encoding**: Kaspa's native address format that combines a network prefix, version byte, and payload with error-detecting checksums

- **Address Versions**: Three standard address types—PubKey (Schnorr), PubKeyECDSA (ECDSA compatibility), and ScriptHash (complex spending conditions)

- **Pay-to-Public-Key **(P2PK): Kaspa's default payment method that embeds the public key directly in the script, avoiding the hash layer used by Bitcoin's P2PKH

- **The Kaspa Script System**: A stack-based script engine that defines spending conditions through opcodes and data manipulation

- **Pay-to-Script-Hash **(P2SH): Enables complex spending conditions like multisig by encoding scripts as Blake2b hashes

- **Collision Resistance and Hash Function Security**: How Kaspa's use of Blake2b with 256-bit output provides superior collision resistance compared to Bitcoin's HASH160 approach

- **HD Wallets and Key Derivation**: BIP32-based hierarchical deterministic wallets with Kaspa's custom derivation path for receive/change address chains

These cryptographic concepts work together to provide the security and privacy that make Kaspa function as a decentralized payment system. In the next chapter, we'll explore how wallets implement these cryptographic concepts to provide a user-friendly interface for managing funds, backing up keys, and interacting with the Kaspa network. We'll learn how wallets protect your private keys, track your balance, and construct transactions that move your kaspa securely.
