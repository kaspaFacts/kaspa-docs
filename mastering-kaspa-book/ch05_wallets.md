# Wallet Recovery

Alice has received kaspa to her wallet, but if she loses access to her private keys, those kaspa will be forever unspendable. Unlike a physical wallet that holds cash, a Kaspa wallet doesn't actually contain kaspa—it contains the cryptographic keys that prove ownership of kaspa recorded on the Kaspa DAG. Losing these keys is equivalent to losing the kaspa itself, with no way to recover them.

Wallet and protocol developers have worked for years to design systems that allow users to recover access to their funds after a problem without compromising security the rest of the time. In this chapter, we'll examine the different methods employed by wallets to prevent the loss of data from becoming a loss of money.

Some solutions have almost no downsides and are universally adopted by modern wallets. We'll recommend these as best practices. Other solutions have both advantages and disadvantages, leading different wallet authors to make different trade-offs. In those cases, we'll describe the various options available so you can make informed decisions about protecting your kaspa.

## Independent Key Generation

Wallets for physical cash hold that cash, so it's unsurprising that many people mistakenly believe that Kaspa wallets contain kaspa. In fact, what many people call a Kaspa wallet—which we call a wallet database to distinguish it from wallet applications—contains only keys. Those keys are associated with kaspa recorded on the blockchain. By proving to Kaspa full nodes that you control the keys, you can spend the associated kaspa.

Simple wallet databases contain both the public keys to which kaspa are received and the private keys that allow creating the signatures necessary to authorize spending those kaspa. Other wallets' databases may contain only public keys, or only some of the private keys necessary to authorize a spending transaction. Their wallet applications produce the necessary signatures by working with external tools, such as hardware signing devices or other wallets in a multisignature scheme.

It's possible for a wallet application to independently generate each of the wallet keys it later plans to use. All early Bitcoin wallet applications did this, but it required users to back up the wallet database each time they generated and distributed new keys, which could be as often as each time they generated a new address to receive a new payment. Failure to back up the wallet database on time would lead to the user losing access to any funds received to keys that had not been backed up.

For each independently generated key, the user would need to back up about 32 bytes, plus overhead. Some users and wallet applications tried to minimize the amount of data that needed to be backed up by only using a single key. Although that can be secure, it severely reduces the privacy of that user and all of the people with whom they transact. People who valued their privacy and those of their peers created new key pairs for each transaction, producing wallet databases that could only reasonably be backed up using digital media.

Modern wallet applications don't independently generate keys but instead derive them from a single random seed using a repeatable (deterministic) algorithm.

## Deterministic Key Generation

A hash function will always produce the same output when given the same input, but if the input is changed even slightly, the output will be different. If the function is cryptographically secure, nobody should be able to predict the new output--not even if they know the new input.

This allows us to take one random value and transform it into a practically unlimited number of seemingly random values. Even more useful, later using the same hash function with the same input (called a seed) will produce the same seemingly random values:

```
# Collect some entropy (randomness)
$ dd if=/dev/random count=1 status=none | sha256sum
f1cc3bc03ef51cb43ee7844460fa5049e779e7425a6349c8e89dfbb0fd97bb73  -

# Set our seed to the random value
$ seed=f1cc3bc03ef51cb43ee7844460fa5049e779e7425a6349c8e89dfbb0fd97bb73

# Deterministically generate derived values
$ for i in {0..2} ; do echo "$seed + $i" | sha256sum ; done
50b18e0bd9508310b8f699bad425efdf67d668cb2462b909fdb6b9bd2437beb3  -
a965dbcd901a9e3d66af11759e64a58d0ed5c6863e901dfda43adcd5f8c744f3  -
19580c97eb9048599f069472744e51ab2213f687d4720b0efc5bb344d624c3aa  -
```

If we use the derived values as our private keys, we can later generate exactly those same private keys by using our seed value with the algorithm we used before. A user of deterministic key generation can back up every key in their wallet by simply recording their seed and a reference to the deterministic algorithm they used. For example, even if Alice has a million kaspa received to a million different addresses, all she needs to back up in order to later recover access to those kaspa is:

```
f1cc 3bc0 3ef5 1cb4 3ee7 8444 60fa 5049
e779 e742 5a63 49c8 e89d fbb0 fd97 bb73
```

## Public Child Key Derivation

The deterministic key generation method described above has a significant limitation: it requires the private key to generate child keys. This means that if you want to generate public keys (for receiving payments) on a public-facing system like a web server, you would need to expose your private key to that system--a major security risk.

Kaspa, like Bitcoin, uses a more sophisticated approach called Hierarchical Deterministic (HD) key derivation based on BIP32. This allows you to derive child public keys from a parent public key without needing the corresponding private key. This is made possible through the mathematics of elliptic curve cryptography.

In Kaspa, we use the secp256k1 elliptic curve. The relationship between a private key (k) and its corresponding public key (K) is defined by:

```
K = k × G
```

Where G is the generator point of the curve. This operation is elliptic curve point multiplication, not ordinary multiplication. The critical property is that while calculating K from k is straightforward, deriving k from K is computationally infeasible.

HD key derivation uses a special technique that allows deriving child public keys from parent public keys using only the parent public key and a chain code. The chain code is a 256-bit random value that introduces entropy into the derivation process. This means:

- You can derive an unlimited tree of public keys from a single extended public key (xpub)
- The xpub can be safely shared with or stored on insecure systems
- Private keys remain on a separate, secure device
- The secure device only needs to be used when signing transactions

This separation is particularly useful for hardware signing devices and web servers. A web server can hold an extended public key and generate unique addresses for each customer order, improving privacy, while the private keys remain on a hardware wallet that is only connected when funds need to be spent.

Kaspa's implementation of HD key derivation uses custom prefixes for extended keys: kpub for extended public keys and kprv for extended private keys, distinguishing them from Bitcoin's xpub and xprv prefixes.

## Hierarchical Deterministic (HD) Key Generation (BIP32)

The deterministic key generation method described above is a significant improvement over independent key generation, but it still has limitations. All keys are derived from a single seed in a linear sequence, which doesn't provide the organizational flexibility needed for complex use cases.

Kaspa, like Bitcoin, uses Hierarchical Deterministic (HD) key generation based on BIP32. This standard allows you to derive a tree of keys from a single seed, with each branch of the tree serving a different purpose. This tree structure provides enormous flexibility while maintaining the security benefits of deterministic key generation.

In an HD wallet, you can have separate branches for different purposes:

- **Receive addresses**: A branch for generating addresses where you receive payments
- **Change addresses**: A separate branch for generating addresses used when sending change back to yourself
- **Multiple accounts**: Different branches for different accounting purposes (personal, business, etc.)
- **Corporate use cases**: Separate branches for different departments or projects

The tree structure is defined by a derivation path, which specifies how to navigate from the master seed to a specific key. Each level of the tree can have up to 2 billion child keys, and the tree can be as deep as needed.

The key insight of HD wallets is that you can back up a single seed and recover an entire tree of keys. This means you can have millions of addresses across multiple accounts, all recoverable from a single backup phrase.

Kaspa's implementation of HD wallets uses the standard BIP32 specification with custom derivation paths optimized for the Kaspa network. The wallet framework automatically manages the tree structure, generating new addresses as needed and tracking which addresses have been used.

For a standard single-signature wallet, the derivation path follows the pattern:

```
m/44'/111111'/<account>'/<address_type>
```

Where:

- **m** is the master key
- **44'** indicates the purpose (BIP44 for single-signature wallets)
- **111111'** is Kaspa's registered coin type
- **<account>'** is the account index (hardened derivation)
- **<address_type>** is 0 for receive addresses or 1 for change addresses

This structure allows you to have multiple accounts (for different purposes) with separate receive and change address chains, all derived from a single seed.

## Seeds and Recovery Codes

Kaspa wallets use BIP39 recovery codes to represent the entropy required for a wallet's root seed. A recovery code is a sequence of words that encodes a random seed, which is then used to generate a master extended private key (kprv). The mnemonic is your backup: anyone with access to it can recover your entire wallet.

### Mnemonic Seed Phrases

Kaspa wallets typically start with a mnemonic phrase—a sequence of 12 or 24 words that encodes a random seed. The number of words corresponds to the amount of entropy used:

- **12 words**: 128 bits of entropy
- **24 words**: 256 bits of entropy

The wallet generates these words automatically using a standardized process defined in BIP39. The wallet starts from a source of entropy, adds a checksum, and then maps the entropy to a word list from a predefined dictionary of 2,048 words.

From the mnemonic, the wallet derives a master key using BIP32 standards. Kaspa uses the kprv prefix for extended private keys and kpub for extended public keys.

### Recovery Code Types in Kaspa

Kaspa supports different recovery code schemes depending on the wallet generation:

- **BIP39**: Standard mnemonic phrases with optional passphrase support
- **Gen0 Legacy**: Older Kaspa wallets using the '972 derivation path (deprecated)
- **Gen1 Standard**: Current standard using the '111111' derivation path

New wallets should use Gen1 derivation with BIP39 recovery codes. The wallet framework automatically handles the correct derivation based on the account type.

### Recovery Code Passphrases

BIP39 supports an optional passphrase that serves as an additional security factor. When you create a recovery code, you can optionally add a passphrase that will be required along with the mnemonic phrase to recover your wallet.

The passphrase is used as a salt in the key-stretching function that converts the mnemonic into the seed. This means:

- The same mnemonic phrase with different passphrases will produce completely different wallets
- An attacker who obtains your mnemonic phrase still cannot access your funds without the passphrase
- If you forget your passphrase, you cannot recover your wallet even with the mnemonic phrase

This provides a trade-off between additional security and the risk of loss. Some users use passphrases for plausible deniability (a "duress wallet" with a smaller amount of funds), while others avoid them to eliminate the risk of forgetting the passphrase.

### Risks of Memorizing Recovery Codes

Some users attempt to memorize their recovery codes instead of writing them down. This is strongly discouraged for several reasons:

- Human memory is unreliable and can degrade over time
- Stress or trauma can cause memory loss
- You may forget the exact word order or spelling
- There is no way to recover a forgotten mnemonic phrase

Recovery codes should always be written down on paper or metal and stored in a secure location. Never store them digitally in plain text, and never share them with anyone.

## Backing Up Nonkey Data

While keys are the most critical data to back up, wallets also store other important information that users may want to preserve. This nonkey data doesn't directly control access to funds, but losing it can be inconvenient and may have tax or accounting implications.

### Transaction Labels and Metadata

Many wallet applications allow users to add labels to transactions or addresses. These labels might include:

- Transaction descriptions (e.g., "Rent payment," "Coffee shop")
- Contact names associated with addresses
- Category tags for expense tracking
- Notes about specific transactions

This metadata is stored locally in the wallet database and is not recorded on the Kaspa network. If you lose your wallet database without backing up this information, you'll lose access to these labels even though you can still recover your funds using your recovery code.

### Exchange Rate Information

Some wallets track the fiat currency value of transactions at the time they occur. This information can be valuable for:

- Tax reporting and capital gains calculations
- Personal financial tracking
- Historical analysis of spending patterns

Like labels, exchange rate data is stored locally and not on the blockchain. Losing this data may make tax preparation more difficult, though it doesn't affect your ability to spend your kaspa.

### Backup Strategies for Nonkey Data

To preserve nonkey data, consider these approaches:

- **Regular wallet database backups**: Export your entire wallet database periodically
- **Label export/import**: Some wallets support exporting labels to a separate file
- **Cloud synchronization**: Use encrypted cloud backup for wallet databases
- **Multiple backup locations**: Keep copies in different physical locations

Kaspa wallets support encrypted storage using XChaCha20Poly1305 encryption, which protects your data both at rest and during backup operations.

## Backing Up Key Derivation Paths

When backing up a wallet, it's important to understand not just what to back up, but also how the keys are derived. The derivation path—the sequence of steps used to generate keys from your seed—is essential information for wallet recovery.

### Implicit vs Explicit Derivation Paths

There are two approaches to backing up derivation information:

- **Implicit paths**: The wallet uses a standard, well-known derivation scheme. As long as you have your recovery code and know the standard scheme, you can recover your wallet without explicitly recording the derivation path.
- **Explicit paths**: The wallet uses a custom or non-standard derivation scheme that must be recorded separately. Without this information, even with your recovery code, you may not be able to derive the correct keys.

### Kaspa's Standard Derivation Paths

Kaspa uses implicit derivation paths based on BIP44 with a custom coin type. For most users, no additional derivation path information needs to be backed up beyond the recovery code itself.

For single-signature wallets, the standard path is:

```
m/44'/111111'/<account>'/<address_type>
```

For multisig wallets, the standard path is:

```
m/45'/111111'/<account>'/<cosigner>/<address_type>
```

These paths are built into the wallet software and will be used automatically during recovery.

### Gen0 vs Gen1 Derivation

Kaspa supports two derivation schemes:

- **Gen0 **(Legacy): Uses the '972 coin type for compatibility with older wallets
- **Gen1 **(Standard): Uses the '111111' coin type, the current standard

When recovering a wallet, the wallet software will typically attempt both schemes or allow you to specify which one to use. New wallets should always use Gen1.

### Multisig Considerations

For multisig wallets, additional information may need to be backed up:

- **Cosigner public keys**: The extended public keys (xpubs) of all cosigners
- **Required signature count**: The number of signatures needed (M-of-N)
- **Derivation path**: The specific path used for each cosigner

This information is often encoded in a wallet descriptor or configuration file that should be backed up alongside your recovery code.

## Kaspa Wallet Technology Stack

Kaspa wallets are built on a foundation of well-established cryptographic standards combined with Kaspa-specific optimizations. This technology stack ensures compatibility with existing tools while providing the security and performance needed for the Kaspa network.

The core components of the Kaspa wallet technology stack include:

- **BIP39**: The standard for mnemonic phrase generation and seed derivation. Kaspa uses the standard BIP39 wordlist and checksum algorithm, ensuring compatibility with other BIP39 wallets while supporting Kaspa's custom derivation paths.
- **BIP32**: Hierarchical Deterministic (HD) key derivation. Kaspa implements the full BIP32 specification, allowing for the derivation of an unlimited tree of keys from a single seed. This enables features like separate receive and change address chains, multiple accounts, and complex organizational structures.
- **Kaspa Custom Derivation Paths**: While BIP32 provides the framework, Kaspa uses custom derivation paths optimized for the network. The standard path uses coin type 111111' (Gen1) instead of Bitcoin's 0', with purpose values of 44 for single-signature and 45 for multisig wallets.
- **Schnorr Signatures**: Kaspa uses Schnorr signatures as the default signing scheme, with ECDSA support for compatibility. Schnorr signatures offer improved security properties and efficiency compared to ECDSA.
- **Bech32 Address Encoding**: Kaspa uses bech32 as its native address format, providing human-readable addresses with built-in error detection. The format includes a network prefix (e.g., kaspa: for mainnet), a version byte indicating the address type, and the payload.
- **Encrypted Storage**: Wallet data is encrypted using XChaCha20Poly1305, a modern authenticated encryption algorithm. This protects sensitive data like private keys and mnemonics both at rest and during backup operations.

This combination of standards and customizations allows Kaspa wallets to benefit from the extensive tooling and testing of established standards while optimizing for the specific requirements of the Kaspa network.

## BIP39 Recovery Codes in Kaspa

Kaspa wallets implement the BIP39 standard for recovery codes, providing a human-readable backup mechanism that works across different wallet implementations. The BIP39 process converts random entropy into a sequence of words that can later be used to reconstruct the wallet's seed.

### Generating a Recovery Code

Recovery codes are generated automatically by the wallet application using the standardized process defined in BIP39. The wallet starts from a source of entropy, adds a checksum, and then maps the entropy to a word list:

1. Create a random sequence (entropy) of 128 to 256 bits.
2. Create a checksum of the random sequence by taking the first (entropy-length/32) bits of its SHA256 hash.
3. Add the checksum to the end of the random sequence.
4. Split the result into 11-bit length segments.
5. Map each 11-bit value to a word from the predefined dictionary of 2,048 words.
6. The recovery code is the sequence of words.

The relationship between the size of the entropy data and the length of recovery code in words is:

| Entropy (bits) | Checksum (bits) | Entropy + Checksum (bits) | Recovery code words |
|----------------|-----------------|---------------------------|---------------------|
| 128            | 4               | 132                       | 12                  |
| 160            | 5               | 165                       | 15                  |
| 192            | 6               | 198                       | 18                  |
| 224            | 7               | 231                       | 21                  |
| 256            | 8               | 264                       | 24                  |

### From Recovery Code to Seed

The recovery code represents entropy with a length of 128 to 256 bits. The entropy is then used to derive a longer (512-bit) seed through the use of a key-stretching function. The seed produced is then used to build a deterministic wallet and derive its keys.

The key-stretching function takes two parameters: the entropy and a salt. The purpose of a salt in a key-stretching function is to make it difficult to build a lookup table enabling a brute-force attack. In the BIP39 standard, the salt has another purpose--it allows the introduction of a passphrase that serves as an additional security factor protecting the seed.

The process continues as follows:

7. The first parameter to the PBKDF2 key-stretching function is the entropy produced from step 6.
8. The second parameter to the PBKDF2 key-stretching function is a salt. The salt is composed of the string constant "mnemonic" concatenated with an optional user-supplied passphrase string.
9. PBKDF2 stretches the recovery code and salt parameters using 2,048 rounds of hashing with the HMAC-SHA512 algorithm, producing a 512-bit value as its final output. That 512-bit value is the seed.

The key-stretching function, with its 2,048 rounds of hashing, makes it slightly harder to brute-force attack the recovery code using software. Special-purpose hardware is not significantly affected. For an attacker who needs to guess a user's entire recovery code, the length of the code (128 bits at a minimum) provides more than sufficient security. But for cases where an attacker might learn a small part of the user's code, key-stretching adds some security by slowing down how fast an attacker can check different recovery code combinations.

### Entropy Requirements

BIP32 allows seeds to be from 128 to 512 bits. BIP39 accepts from 128 to 256 bits of entropy. The variation in these numbers makes it unclear how much entropy is needed for safety.

BIP32 extended private keys consist of a 256-bit key and a 256-bit chain code, for a total of 512 bits. That means there's a maximum of 2^512 different possible extended private keys. If you start with more than 512 bits of entropy, you'll still get an extended private key containing 512 bits of entropy--so there's no point in using more than 512 bits even if any of the standards we mentioned allowed that.

However, even though there are 2^512 different extended private keys, there are only (slightly less than) 2^256 regular private keys--and it's those private keys that actually secure your kaspa. That means, if you use more than 256 bits of entropy for your seed, you still get private keys containing only 256 bits of entropy. There may be future improvements to elliptic curve cryptography that increase the security level, but for secp256k1, 128 bits of security is the practical maximum.

For most users, 128 bits of entropy (12 words) provides more than sufficient security. Higher entropy values (24 words) provide protection against future cryptographic advances and against partial exposure of the recovery code, but the additional security is diminishing returns for most use cases.

### Optional Passphrase in BIP39

BIP39 supports an optional passphrase that can be added to the recovery code. This passphrase is used as part of the salt in the key-stretching function, meaning that the same recovery code with different passphrases will produce completely different seeds and therefore completely different wallets.

An important property of BIP39 passphrases is that there is no "wrong" passphrase. Any passphrase you provide will produce a valid seed and therefore a valid wallet. This property enables a feature called plausible deniability or "duress wallets"--you can have a wallet with a small amount of kaspa protected by one passphrase, and a larger wallet protected by a different passphrase. If forced to reveal your passphrase, you can reveal the one for the smaller wallet.

However, this property also creates a significant risk: if you forget your passphrase, you cannot recover your wallet even with the correct recovery code. The passphrase is not stored anywhere and cannot be recovered. This makes BIP39 passphrases a double-edged sword: they provide additional security and plausible deniability, but they also add another point of failure.

For most users, the risk of forgetting a passphrase outweighs the benefits. If you do choose to use a passphrase, it's critical to have a reliable backup plan that includes both the recovery code and the passphrase, and to regularly test that you can recover your wallet using both.

## Creating an HD Wallet from the Seed

The root seed is a 128-, 256-, or 512-bit random number that serves as the foundation for the entire HD wallet tree. From this seed, we derive the master extended private key (kprv), which is the root of the key derivation tree.

The process begins by applying HMAC-SHA512 to the seed. The output of this hash function is split into two parts:

- The first 256 bits become the master private key
- The remaining 256 bits become the master chain code

The master private key is then used to generate the master public key through elliptic curve multiplication on the secp256k1 curve:

```
K = k × G
```

Where k is the master private key and G is the generator point. The resulting K is the master public key.

The master extended private key (kprv) consists of three components:

- The master private key (256 bits)
- The master chain code (256 bits)
- Additional metadata including the derivation path and version information

Similarly, the master extended public key (kpub) consists of:

- The master public key (derived from the private key)
- The master chain code (same as the private key's chain code)
- The same metadata

This structure allows the extended public key to derive child public keys without access to the private key, while the chain code provides the entropy needed for the derivation process.

Once you have the master extended keys, you can derive the entire tree of child keys using the BIP32 derivation process. Each child key derivation uses the parent key, the parent chain code, and an index number to produce a new key and chain code pair.

For Kaspa wallets, the master extended private key is typically represented with the kprv prefix (for mainnet) or ktrv prefix (for testnet), while the master extended public key uses the kpub or ktub prefix respectively. These custom prefixes distinguish Kaspa extended keys from Bitcoin's xprv/xpub format.

## Extended Keys

Extended keys are a fundamental concept in HD wallets that enable the hierarchical derivation of child keys. An extended key contains not just a cryptographic key (either private or public), but also additional metadata needed for the derivation process.

An extended key consists of three main components:

- **The key itself**: Either a private key (for extended private keys) or a public key (for extended public keys)
- **The chain code**: A 256-bit random value that provides entropy for child key derivation
- **Metadata**: Including the depth in the derivation tree, parent fingerprint, child number, and version information

The chain code is particularly important because it introduces entropy into the derivation process. Without it, an attacker who obtained a parent public key might be able to derive child public keys more easily. The chain code ensures that each derivation step incorporates additional randomness.

Extended keys come in two types:

- **Extended Private Keys **(xprv): Contain a private key and can derive both child private keys and child public keys. These must be kept secret as they provide full control over all derived keys.
- **Extended Public Keys **(xpub): Contain only a public key and can derive only child public keys. These can be safely shared to create watch-only wallets or to generate receiving addresses on insecure systems.

In Kaspa, extended keys use custom prefixes to distinguish them from Bitcoin extended keys. The kprv prefix indicates a Kaspa extended private key, while kpub indicates a Kaspa extended public key. For testnet, the prefixes are ktrv and ktub respectively.

## HD Wallet Key Identifier (Path)

The HD wallet tree structure is navigated using a derivation path, which specifies the sequence of steps to reach a specific key from the master key. The path is written as a sequence of numbers separated by slashes, with each number representing a child index.

The path notation follows these conventions:

- The path starts with **m** for private keys derived from the master private key
- The path starts with **M** for public keys derived from the master public key
- Each level is separated by a forward slash (/)
- Hardened derivation is indicated by a prime symbol (') or the letter h after the index
- Normal derivation uses the index without a prime symbol

For example:

- `m/0` - The first child private key of the master private key
- `m/0/0` - The first child of the first child
- `m/0'/0` - The first normal child of the first hardened child
- `M/23/17/0/0` - A public key derived from the master public key

The ancestry is read from right to left: the rightmost number is the most recent derivation step, and the leftmost m or M represents the master key.

Each parent key can have up to 2 billion normal children (indices 0 to 2^31-1) and up to 2 billion hardened children (indices 2^31 to 2^32-1). This allows for an enormous number of possible keys at each level of the tree.

The tree can be infinitely deep, limited only by practical considerations. In practice, most wallets use a depth of 5 or 6 levels, which provides more than enough organizational flexibility while keeping the derivation paths manageable.

## Navigating HD Wallet Tree Structure

The HD wallet tree structure is organized according to BIP43 and BIP44 standards, which provide a systematic way to navigate the tree. BIP43 proposes using the first hardened child index to identify the "purpose" of the tree structure, while BIP44 specifies a multiaccount hierarchy for cryptocurrency wallets.

In the BIP44 standard, the first three levels of the tree are mandatory and follow this pattern:

- **Purpose**: The first level identifies the purpose of the tree structure. For HD wallets following BIP44, this is always 44'. For multisig wallets, Kaspa uses purpose 45'.
- **Coin Type**: The second level identifies the cryptocurrency. Bitcoin uses 0' for mainnet and 1' for testnet. Kaspa uses 111111' for its registered coin type (Gen1 standard) or 972' for legacy wallets (Gen0).
- **Account**: The third level allows users to create multiple accounts within the same wallet. Each account is the root of its own subtree, enabling logical separation of funds for different purposes (personal, business, etc.).

The fourth level of the tree is "change," which has two subtrees: one for receiving addresses and one for change addresses. Unlike the previous levels which use hardened derivation, this level uses normal derivation. This allows this level of the tree to export extended public keys for use in nonsecured environments.

The fifth level is the "address\_index," which is used to generate specific addresses. For example, the third receiving address for payments in the primary account would be m/44'/111111'/0'/0/2.

Each parent key can have up to 2 billion normal children (indices 0 to 2^31-1) and up to 2 billion hardened children (indices 2^31 to 2^32-1). This allows for an enormous number of possible keys at each level of the tree. The tree can be infinitely deep, limited only by practical considerations.

## Kaspa's Custom HD Wallet Structure

Kaspa adapts the BIP44 standard with custom coin types and purpose values optimized for the Kaspa network. This ensures compatibility with existing HD wallet infrastructure while providing clear identification of Kaspa keys.

For single-signature wallets, the standard Kaspa derivation path is:

```
m/44'/111111'/<account>'/<address_type>
```

For multisig wallets, the derivation path includes a cosigner index:

```
m/45'/111111'/<account>'/<cosigner>/<address_type>
```

The path components are:

- **Purpose**: 44 for single-signature wallets, 45 for multisig wallets
- **Coin Type**: 111111' for Gen1 (standard), 972' for Gen0 (legacy)
- **Account**: Account index (0, 1, 2, ...) using hardened derivation
- **Cosigner**: Cosigner index for multisig (0, 1, 2, ...) - only present in multisig paths
- **Address Type**: 0 for receive addresses, 1 for change addresses

The wallet framework automatically manages these paths based on the account type. When creating a new account, the framework uses the appropriate purpose and coin type, and when deriving addresses, it uses the correct address\_type value.

For a standard single-signature wallet, the first receive address uses the path m/44'/111111'/0'/0/0, while the first change address uses m/44'/111111'/0'/1/0. This structure allows for multiple accounts with separate receive and change chains, all derived from a single seed.

The derivation path is built dynamically by the build\_derivate\_path function, which checks whether the wallet is multisig and conditionally includes the cosigner index only when needed.

## Extended Key Prefixes in Kaspa

Extended keys use specific prefixes to identify the key type and network. These prefixes are encoded in the extended key serialization and allow wallet software to distinguish between different key types and networks at a glance.

Kaspa uses custom prefixes to distinguish its extended keys from Bitcoin's standard prefixes:

| Prefix | Description | Version |
|--------|-------------|---------|
| kprv   | Kaspa extended private key (mainnet) | 0x038f2ef4 |
| kpub   | Kaspa extended public key (mainnet) | 0x038f332e |
| ktrv   | Kaspa extended private key (testnet) | 0x03909e07 |
| ktub   | Kaspa extended public key (testnet) | 0x0390a241 |

These custom prefixes prevent confusion between Kaspa and Bitcoin extended keys, which is important since both networks use the same underlying BIP32 standard. The prefixes are 4-character ASCII strings that are Base58Check-encoded along with the key data.

For comparison, Bitcoin uses:

- **xprv/xpub**: Bitcoin mainnet extended private/public keys
- **tprv/tpub**: Bitcoin testnet extended private/public keys

When importing or exporting extended keys, it's critical to use the correct prefix for the network and cryptocurrency. Using a Bitcoin xprv with Kaspa wallet software will result in incorrect address derivation, and vice versa.

## Summary

In this chapter, you learned how Kaspa wallets use hierarchical deterministic key generation and recovery codes to prevent the loss of funds from becoming permanent. We explored the systems that modern wallet applications use to help you protect your kaspa.

We covered:

- **Independent Key Generation**: The early approach of generating each key independently, which required frequent backups and had privacy limitations

- **Deterministic Key Generation**: Using hash functions to derive unlimited keys from a single seed, enabling simple backup strategies

- **Public Child Key Derivation**: The mathematical foundation that allows deriving child public keys from parent public keys without private keys, enabling watch-only wallets and secure address generation on insecure systems

- **Hierarchical Deterministic (HD) Key Generation **(BIP32): The tree-based key derivation system that provides organizational flexibility while maintaining the security benefits of deterministic wallets

- **Seeds and Recovery Codes**: BIP39 mnemonic phrases that encode wallet entropy in human-readable form, with support for 12-word (128-bit) and 24-word (256-bit) phrases

- **Gen0 vs Gen1 Derivation**: Kaspa's legacy Gen0 scheme using '972 coin type and the current Gen1 standard using '111111' coin type

- **Backing Up Nonkey Data**: The importance of backing up transaction labels, metadata, and other wallet data that doesn't directly control funds but may be valuable for accounting and tax purposes

- **Backing Up Key Derivation Paths**: Understanding implicit vs explicit derivation paths and the additional information needed for multisig wallet recovery

- **Kaspa Wallet Technology Stack**: The combination of BIP39, BIP32, custom derivation paths, Schnorr signatures, bech32 encoding, and encrypted storage that forms the foundation of Kaspa wallets

- **Extended Keys**: The structure of extended keys containing the key itself, chain code, and metadata needed for hierarchical derivation

- **HD Wallet Key Identifier **(Path): The notation system for navigating the HD wallet tree using derivation paths like m/44'/111111'/0'/0/0

- **Navigating HD Wallet Tree Structure**: The BIP43/BIP44 standards that organize the tree into purpose, coin type, account, change, and address index levels

- **Kaspa's Custom HD Wallet Structure**: Kaspa's specific implementation with purpose values 44 (single-sig) and 45 (multisig), coin type 111111', and optional cosigner index for multisig wallets

- **Extended Key Prefixes**: Kaspa's custom prefixes (kprv/kpub/ktrv/ktub) that distinguish Kaspa extended keys from Bitcoin's standard prefixes

These wallet recovery systems work together to provide the security and usability that make Kaspa function as a decentralized payment system. In the next chapter, we'll explore how wallets implement transactions to move your kaspa securely across the network.
