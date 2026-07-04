# Contributing to Mastering Kaspa

## Overview

This project is an attempt at "kaspafying" the excellent [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook) book by Andreas M. Antonopoulos, adapting it for the Kaspa cryptocurrency ecosystem.

**Status: Alpha / Prerelease** 🚧

This book is currently in early development and open to contributors who want to help bring accurate, comprehensive documentation about Kaspa to the community.

---

## What This Book Is (and Isn't)

### ✅ What this IS:
- A faithful adaptation of Mastering Bitcoin's structure and approach
- Technical documentation for Kaspa developers and enthusiasts
- An educational resource explaining Kaspa's unique architecture (blockDAG, GHOSTDAG consensus)
- Community-driven open source content

### ❌ What this is NOT:
- An official Kaspa Foundation publication
- A replacement for the original Mastering Bitcoin
- Financial advice or investment recommendations

---

## How You Can Contribute

### 1. Content Contributions

**Chapter Writing:** Help convert remaining chapters from Bitcoin to Kaspa terminology and concepts.

**Technical Review:** If you have expertise in:
- BlockDAG data structures
- GHOSTDAG/DAGKNIGHT consensus algorithms
- Kaspa node implementation
- Cryptography and distributed systems

Please review technical accuracy of content!

### 2. Bug Reports & Corrections

Found an error? Please open an issue with:
- Chapter/section reference
- Description of the issue
- Suggested correction (if applicable)
- Source/reference if available

### 3. Improvements

Suggestions for:
- Better explanations of complex topics
- Additional examples or diagrams
- Clarifications on Kaspa-specific concepts

---

## Technical Guidelines

### Kaspa Fact Sheet

When contributing, apply these conversions consistently:

| Bitcoin | Kaspa |
|---------|-------|
| 10 minutes block time | **0.1 seconds (100ms)** |
| 21 million max supply | **28.7 billion** |
| Halving every 4 years | **Halving every year** (smooth monthly steps) |
| blockchain | **blockDAG** |
| SPV clients | Not applicable (all nodes pruned by default) |
| satoshis | **sompi** (1 kaspa = 100,000,000 sompi) |

### Key Kaspa Concepts to Emphasize

- **No L2 needed**: Base layer handles all transactions natively
- **Full nodes pruned by default**: UTXO set commitment in headers
- **Sub-network-latency block production**: Throughput limited only by physics
- **GHOSTDAG consensus**: Preserves all valid blocks, topological ordering

### Writing Style

- Use clear, accessible language (following Mastering Bitcoin's approach)
- Explain concepts before diving into technical details
- Use examples and analogies where helpful
- Maintain neutral, educational tone
- Avoid hype or promotional language

---

## Getting Started

1. **Fork** the repository
2. **Clone** your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/mastering-kaspa-book.git
   cd kaspa-book
   ```
3. Create a branch for your contribution:
   ```bash
   git checkout -b feature/chapter-description
   ```
4. Make your changes to the `.md` files
5. **Commit** with descriptive messages
6. **Push** and create a Pull Request

---

## Chapter Status

### Published Chapters (Ready for Review)

| Chapter | File | Lines | Status | Description |
|---------|------|-------|--------|-------------|
| 1 | ch01_intro.md | ~270 | ✅ PUBLISHED | Introduction to Kaspa, wallets, getting started |
| 2 | ch02_overview.md | ~188 | ✅ PUBLISHED | How Kaspa works, transactions, mining overview |
| 3 | ch03_kaspa-node.md | ~756 | ✅ PUBLISHED | Running a Kaspa node, configuration, RPC API |
| 4 | ch04_keys.md | ~504 | ✅ PUBLISHED | Keys, addresses, cryptography, HD wallets |

### In Progress / Not Started

| Chapter | Status | Notes |
|---------|--------|------|
| ch05+ | 🚧 Pending | |

---

**Progress**: 4 chapters published (~1,718 lines total). Ready for community review and feedback!

---

## Questions?

- Open an issue for discussion
- Check existing issues for ongoing conversations
- Review the PHANTOM GHOSTDAG paper for technical details:
  - Authors: Yonatan Sompolinsky, Shai Wyborski, Aviv Zohar
  - Published: November 2021

---

## License

This adaptation is released under the same license as Mastering Bitcoin.

Original work: © Andreas M. Antonopoulos  
Adaptation: © Kaspa Community Contributors

---

**Thank you for contributing to open source documentation for Kaspa!** 
