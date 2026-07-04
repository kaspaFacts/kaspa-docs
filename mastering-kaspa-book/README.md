# Mastering Kaspa 📘

## A Comprehensive Guide to the Kaspa Cryptocurrency

---

## About This Book

**Mastering Kaspa** is a comprehensive technical guide to the Kaspa cryptocurrency, adapted from the acclaimed "Mastering Bitcoin" by Andreas M. Antonopoulos.

This book provides:
- 📖 **In-depth explanations** of Kaspa's unique blockDAG architecture
- 🔐 **Cryptography fundamentals** including Schnorr signatures and elliptic curve cryptography
- 💻 **Node operation** guides for running your own Kaspa full node
- 🛠️ **Developer resources** covering scripts, transactions, and the Kaspa protocol
- 🎓 **Educational content** suitable for both beginners and experienced developers

---

## What Makes Kaspa Different?

Kaspa is not just another cryptocurrency—it's a breakthrough in blockchain technology:

| Feature | Bitcoin | Kaspa |
|---------|---------|-------|
| Block Time | 10 minutes | **0.1 seconds (100ms)** |
| Data Structure | Blockchain | **BlockDAG** |
| Consensus | PoW (linear) | **GHOSTDAG** |
| Max Supply | 21 million | **28.7 billion** |
| Halving Schedule | Every 4 years | **Every year** (smooth steps) |

### Key Innovations:
- ⚡ **Sub-second block times** with no tradeoffs to security or decentralization
- 🔷 **BlockDAG structure** preserves all valid blocks instead of discarding competing ones
- 🌐 **Throughput limited only by physics**—not artificial consensus constraints
- 💎 **No Layer 2 needed**—base layer handles all transactions natively

---

## Table of Contents

### Published Chapters

| Chapter | Title | Description |
|---------|-------|-------------|
| [1](ch01_intro.md) | Introduction | What is Kaspa? Getting started, wallets, receiving your first kaspa |
| [2](ch02_overview.md) | How Kaspa Works | Transactions, the blockDAG, mining overview, confirmations |
| [3](ch03_kaspa-node.md) | Running a Kaspa Node | Installation, configuration, RPC API, hardware requirements |
| [4](ch04_keys.md) | Keys and Addresses | Cryptography, addresses, scripts, HD wallets, BIP32 derivation |

### Coming Soon

- Transactions in Depth
- The Kaspa Network Protocol
- Mining and Consensus (GHOSTDAG)
- Advanced Scripting
- And more...

---

## Quick Start

### Reading the Book

You can read this book directly on GitHub, or clone it locally:

Each chapter is a standalone `.md` file that can be opened in any markdown viewer.

### Recommended Reading Order

1. **Chapter 1** - Start here if you're new to Kaspa
2. **Chapter 2** - Understand how transactions flow through the network
3. **Chapter 4** - Learn the cryptography behind addresses and keys
4. **Chapter 3** - Set up your own node for deeper exploration

---

## For Developers

### Key Technical Concepts Covered

- **Bech32 Address Encoding**: Kaspa's native address format (no Base58check)
- **Schnorr Signatures**: Default signature scheme with 32-byte x-only keys
- **Blake2b Hashing**: Superior collision resistance for P2SH addresses
- **HD Wallet Derivation**:
  - Single-signature: `m/44'/111111'/<account>'/<address_type>`
  - Multisig: `m/45'/111111'/<account>'/<cosigner>/<address_type>`

### Sample Address Format

```
kaspa:qpq64dfdztjjzte7xmqt9ehe4n456xg4v30ekqlveut6u3wfhzyasq9whp8ak
```

---

## Resources

### Official Kaspa Links
- **Website**: https://kaspa.org
- **Documentation**: https://docs.kaspa.org
- **GitHub**: https://github.com/kaspanet
- **BlockDAG Explorer**: https://explorer.kaspa.org

### Technical Papers
- **PHANTOM GHOSTDAG Paper** (2021): The foundation of Kaspa's consensus
  - Authors: Yonatan Sompolinsky, Shai Wyborski, Aviv Zohar

### Related Projects
- **rusty-kaspa**: Reference node implementation
- **Kaspa Wallet Framework**: HD wallet library

---

## Contributing

This is a community-driven project! See [CONTRIBUTING.md](CONTRIBUTING.md) for details on:
- How to submit content contributions
- Technical review guidelines
- Bug reports and corrections

---

## Disclaimer

This book is an educational resource and community project. It is **not**:
- An official Kaspa Foundation publication
- Financial or investment advice
- A substitute for professional consultation

Always do your own research and verify information from multiple sources.

---

## Support the Project

If you find this book helpful:
- ⭐ Star this repository
- 📢 Share with others learning about Kaspa
- 💬 Provide feedback through issues
- 🤝 Contribute improvements or corrections

---

**Happy reading, and welcome to Kaspa!** 🚀
