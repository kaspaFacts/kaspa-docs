# Introduction

Kaspa is a collection of concepts and technologies that form the basis of a digital money ecosystem. Units of currency called kaspa are used to store and transmit value among participants in the Kaspa network. Kaspa users communicate with each other using the Kaspa protocol primarily via the internet, although other transport networks can also be used. The Kaspa protocol stack, available as open source software, can be run on a wide range of computing devices, including laptops and smartphones, making the technology easily accessible.

> **Note:** In this book, the unit of currency is called "kaspa" with a small _k_, and the system is called "Kaspa," with a capital _K_.

Users can transfer kaspa over the network to do just about anything that can be done with conventional currencies, including buying and selling goods, sending money to people or organizations, or extending credit. Kaspa can be purchased, sold, and exchanged for other currencies at specialized currency exchanges. Kaspa is arguably the perfect form of money for the internet because it is fast, secure, and borderless.

Unlike traditional currencies, the kaspa currency is entirely virtual. There are no physical coins or even individual digital coins. The coins are implied in transactions that transfer value from spender to receiver. Users of Kaspa control keys that allow them to prove ownership of kaspa in the Kaspa network. With these keys, they can sign transactions to unlock the value and spend it by transferring it to a new owner. Keys are often stored in a digital wallet on each user's computer or smartphone.

Possession of the key that can sign a transaction is the only prerequisite to spending kaspa, putting the control entirely in the hands of each user.

Kaspa is a distributed, peer-to-peer system. As such, there is no central server or point of control. Units of kaspa are created through a process called "mining," which involves repeatedly performing a computational task that references a list of recent Kaspa transactions. Any participant in the Kaspa network may operate as a miner, using their computing devices to help secure transactions.

**Every 0.1 seconds (100ms), on average**, one Kaspa miner can add security to past transactions and is rewarded with both brand new kaspa and the fees paid by recent transactions. Essentially, Kaspa mining decentralizes the currency-issuance and clearing functions of a central bank and replaces the need for any central bank.

The Kaspa protocol includes built-in algorithms that regulate the mining function across the network. The difficulty of the computational task that miners must perform is adjusted dynamically so that, on average, someone succeeds every 0.1 seconds (100ms) regardless of how many miners (and how much processing) are competing at any moment.

The protocol also periodically decreases the number of new kaspa that are created, limiting the total number of kaspa that will ever be created to a fixed total **of 28.7 billion coins**. The result is that the number of kaspa in circulation closely follows an easily predictable curve where the block reward halves **every year through smooth monthly steps**. At approximately **year 2030**, 99% of all kaspa that will ever exist will have been issued.

Due to Kaspa's diminishing rate of issuance, over the long term, the Kaspa currency is deflationary. Furthermore, nobody can force you to accept any kaspa that were created beyond the expected issuance rate.

---

## Key Innovations

Behind the scenes, Kaspa is also the name of the protocol, a peer-to-peer network, and a distributed computing innovation. Kaspa builds on decades of research in cryptography and distributed systems and includes **five key innovations** brought together in a unique and powerful combination:

| Component | Description |
|-----------|-------------|
| Decentralized peer-to-peer network | The Kaspa protocol |
| Public transaction journal | The blockDAG |
| Validation rules | Consensus rules for independent transaction validation and currency issuance |
| Global consensus mechanism | GHOSTDAG proof-of-work algorithm (with DAGKNIGHT planned for the future) |
| High-throughput design | Ability to produce blocks at sub network latency while preserving security, solving the Bitcoin Scaling Problem |

As a developer, I see Kaspa as akin to the internet of money, a network for propagating value and securing the ownership of digital assets via distributed computation. There's a lot more to Kaspa than first meets the eye.

In this chapter we'll get started by explaining some of the main concepts and terms, getting the necessary software, and using Kaspa for simple transactions. In the following chapters, we'll start unwrapping the layers of technology that make Kaspa possible and examine the inner workings of the Kaspa network and protocol.

---

## Digital Currencies Before Kaspa

The emergence of viable digital money is closely linked to developments in cryptography. This is not surprising when one considers the fundamental challenges involved with using bits to represent value that can be exchanged for goods and services. Three basic questions for anyone accepting digital money are:

1. Can I trust that the money is authentic and not counterfeit?
2. Can I trust that the digital money can only be spent once (known as the "double-spend" problem)?
3. Can I be sure that no one else can claim this money belongs to them and not me?

Issuers of paper money are constantly battling the counterfeiting problem by using increasingly sophisticated papers and printing technology. Physical money addresses the double-spend issue easily because the same paper note cannot be in two places at once.

Of course, conventional money is also often stored and transmitted digitally. In these cases, the counterfeiting and double-spend issues are handled by clearing all electronic transactions through central authorities that have a global view of the currency in circulation.

For digital money, which cannot take advantage of esoteric inks or holographic strips, cryptography provides the basis for trusting the legitimacy of a user's claim to value. Specifically, cryptographic digital signatures enable a user to sign a digital asset or transaction proving the ownership of that asset. With the appropriate architecture, digital signatures also can be used to address the double-spend issue.

When cryptography started becoming more broadly available and understood in the late 1980s, many researchers began trying to use cryptography to build digital currencies. These early digital currency projects issued digital money, usually backed by a national currency or precious metal such as gold.

Although these earlier digital currencies worked, they were centralized and, as a result, were easy to attack by governments and hackers. Early digital currencies used a central clearinghouse to settle all transactions at regular intervals, just like a traditional banking system.

Unfortunately, in most cases these nascent digital currencies were targeted by worried governments and eventually litigated out of existence. Some failed in spectacular crashes when the parent company liquidated abruptly.

To be robust against intervention by antagonists, whether legitimate governments or criminal elements, a _decentralized_ digital currency was needed to avoid a single point of attack. **Kaspa is such a system**, decentralized by design, and free of any central authority or point of control that can be attacked or corrupted.

---

## History of Kaspa

Kaspa was first described in the PHANTOM GHOSTDAG paper, with the network launching in **2021**. The protocol was developed by **Yonatan Sompolinsky, Shai Wyborski, and Aviv Zohar**, building on years of research into blockDAG consensus mechanisms.

Unlike traditional blockchains that discard competing blocks, Kaspa's GHOSTDAG protocol preserves all valid blocks and orders them using a sophisticated topological sorting algorithm, achieving both high throughput and fast finality.

The key insight behind Kaspa is solving the **blockchain trilemma**—the tradeoff between decentralization, security, and scalability. By using a blockDAG structure with 0.1-second block times, Kaspa achieves massive parallelization of transaction processing while maintaining decentralization and security. The protocol allows blocks to be produced at sub-network-latency speeds, meaning the only limit to throughput is the speed of light itself—not artificial constraints imposed by consensus mechanisms.

---

## A Solution to a Distributed Computing Problem

Satoshi Nakamoto's invention is also a practical and novel solution to a problem in distributed computing, known as the "Byzantine Generals' Problem." Briefly, the problem consists of trying to get multiple participants without a leader to agree on a course of action by exchanging information over an unreliable and potentially compromised network.

Satoshi Nakamoto's solution, which uses the concept of proof of work to achieve consensus _without a central trusted authority_, represents a breakthrough in distributed computing.

**Building on this foundation, Kaspa improved the model by scaling the solution without tradeoffs—maintaining Bitcoin's security and decentralization while achieving throughput limited only by network physics rather than artificial constraints.**

---

## Getting Started

Kaspa is a protocol that can be accessed using an application that speaks the protocol. A "Kaspa wallet" is the most common user interface to the Kaspa system, just like a web browser is the most common user interface for the HTTP protocol.

There are many implementations and brands of Kaspa wallets, just like there are many brands of web browsers (e.g., Chrome, Safari, and Firefox). And just like we all have our favorite browsers, Kaspa wallets vary in quality, performance, security, privacy, and reliability.

**There is also a reference implementation of the Kaspa protocol that includes a CLI wallet on the node itself**, which is derived from the original implementation written by the Kaspa development team.

### Choosing a Kaspa Wallet

Kaspa wallets are one of the most actively developed applications in the Kaspa ecosystem. There is intense competition, and while a new wallet is probably being developed right now, several wallets from last year are no longer actively maintained.

Many wallets focus on specific platforms or specific uses and some are more suitable for beginners while others are filled with features for advanced users. Choosing a wallet is highly subjective and depends on the use and user expertise. Therefore, it would be pointless to recommend a specific brand or wallet.

However, we can categorize Kaspa wallets according to their platform and function and provide some clarity about all the different types of wallets that exist. It is worth trying out several different wallets until you find one that fits your needs.

#### Types of Kaspa Wallets

Kaspa wallets can be categorized as follows, according to the platform:

| Type | Description |
|------|-------------|
| **Desktop wallet** | A desktop wallet was the first type of Kaspa wallet created as a reference implementation. Many users run desktop wallets for the features, autonomy, and control they offer. Running on general-use operating systems such as Windows and macOS has certain security disadvantages, however, as these platforms are often insecure and poorly configured. |
| **Mobile wallet** | A mobile wallet is the most common type of Kaspa wallet. Running on smart-phone operating systems such as Apple iOS and Android, these wallets are often a great choice for new users. Many are designed for simplicity and ease-of-use, but there are also fully featured mobile wallets for power users. To avoid downloading and storing large amounts of data, most mobile wallets retrieve information from remote servers, reducing your privacy by disclosing to third parties information about your Kaspa addresses and balances. |
| **Web wallet** | Web wallets are accessed through a web browser and store the user's wallet on a server owned by a third party. This is similar to webmail in that it relies entirely on a third-party server. Some of these services operate using client-side code running in the user's browser, which keeps control of the Kaspa keys in the hands of the user, although the user's dependence on the server still compromises their privacy. Most, however, take control of the Kaspa keys from users in exchange for ease-of-use. It is inadvisable to store large amounts of kaspa on third-party systems. |
| **Hardware signing devices** | Hardware signing devices are devices that can store keys and sign transactions using special-purpose hardware and firmware. They usually connect to a desktop, mobile, or web wallet via USB cable, near-field-communication (NFC), or a camera with QR codes. By handling all Kaspa-related operations on the specialized hardware, these wallets are less vulnerable to many types of attacks. Hardware signing devices are sometimes called "hardware wallets", but they need to be paired with a full-featured wallet to send and receive transactions, and the security and privacy offered by that paired wallet plays a critical role in how much security and privacy the user obtains when using the hardware signing device. |

#### Full Node vs Lightweight

Another way to categorize Kaspa wallets is by their degree of autonomy and how they interact with the Kaspa network:

| Type | Description |
|------|-------------|
| **Full node** | A full node is a program that validates the Kaspa blockDAG and maintains the current state of all unspent transaction outputs (UTXOs). Unlike Bitcoin, Kaspa full nodes are pruned by default—they do not preserve the complete history of every transaction ever made. Instead, each header contains a cryptographic commitment to the UTXO set, linking it directly to the proof-of-work. This architectural choice allows Kaspa to maintain only valid UTXOs while still providing full validation and security, representing a significant efficiency improvement over Bitcoin's model. |
| **Lightweight client** | A lightweight client connects to a full node or other remote server for receiving and sending Kaspa transaction information, but stores the user wallet locally, partially validates the transactions it receives, and independently creates outgoing transactions. |
| **Third-party API client** | A third-party API client is one that interacts with Kaspa through a third-party system of APIs rather than by connecting to the Kaspa network directly. The wallet may be stored by the user or by third-party servers, but the client trusts the remote server to provide it with accurate information and protect its privacy. |

> **Note:** Kaspa is a peer-to-peer (P2P) network. Full nodes are the _peers_: each peer individually validates every confirmed transaction and can provide data to its user with complete authority. Lightweight wallets and other software are _clients_: each client depends on one or more peers to provide it with valid data. Kaspa clients can perform secondary validation on some of the data they receive and make connections to multiple peers to reduce their dependence on the integrity of a single peer, but the security of a client ultimately relies on the integrity of its peers.

#### Who Controls the Keys

A very important additional consideration is _who controls the keys_. As we will see in subsequent chapters, access to kaspa is controlled by "private keys," which are like very long PINs. If you are the only one to have control over these private keys, you are in control of your kaspa. Conversely, if you do not have control, then your kaspa are managed by a third-party who ultimately controls your funds on your behalf.

Key management software falls into two important categories based on control: **wallets**, where you control the keys, and the funds and accounts with custodians where some third-party controls the keys. To emphasize this point, Andreas Antonopoulos coined the phrase:

> _Your keys, your coins. Not your keys, not your coins._

Combining these categorizations, many Kaspa wallets fall into a few groups, with the three most common being:
1. Desktop full node (you control the keys)
2. Mobile lightweight wallet (you control the keys)
3. Web-based accounts with third parties (you don't control the keys)

The lines between different categories are sometimes blurry, as software runs on multiple platforms and can interact with the network in different ways.

---

## Quick Start

Alice is not a technical user and only recently heard about Kaspa from her friend Joe. While at a party, Joe is enthusiastically explaining Kaspa to everyone around him and is offering a demonstration. Intrigued, Alice asks how she can get started with Kaspa.

Joe says that a mobile wallet is best for new users and he recommends a few of his favorite wallets. Alice downloads one of Joe's recommendations and installs it on her phone.

When Alice runs her wallet application for the first time, she chooses the option to create a new Kaspa wallet. Because the wallet she has chosen is a **noncustodial wallet**, Alice (and only Alice) will be in control of her keys. Therefore, she bears responsibility for backing them up, since losing the keys means she loses access to her kaspa.

To facilitate this, her wallet produces a _recovery code_ that can be used to restore her wallet.

### Recovery Codes

Most modern noncustodial Kaspa wallets will provide a recovery code for their user to back up. The recovery code usually consists of numbers, letters, or words selected randomly by the software, and is used as the basis for the keys that are generated by the wallet.

**Sample Recovery Codes:**

| Wallet | Recovery Code |
|--------|---------------|
| BlueWallet | (1) media (2) suspect (3) effort (4) dish (5) album (6) shaft (7) price (8) junk (9) pizza (10) situate (11) oyster (12) rib |
| Electrum | nephew dog crane clever quantum crazy purse traffic repeat fruit old clutch |
| Muun | LAFV TZUN V27E NU4D WPF4 BRJ4 ELLP BNFL |

> **Note:** A recovery code is sometimes called a "mnemonic" or "mnemonic phrase," which implies you should memorize the phrase, but writing the phrase down on paper takes less work and tends to be more reliable than most people's memories. Another alternative name is "seed phrase" because it provides the input ("seed") to the function that generates all of a wallet's keys.

If something happens to Alice's wallet, she can download a new copy of her wallet software and enter this recovery code to rebuild the wallet database of all the on-DAG transactions she's ever sent or received. However, recovering from the recovery code will not by itself restore any additional data Alice entered into her wallet, such as the labels she associated with particular addresses or transactions.

Although losing access to that metadata isn't as important as losing access to money, it can still be important in its own way. Imagine you need to review an old bank or credit card statement and the name of every entity you paid (or who paid you) has been blanked out. To prevent losing metadata, many wallets provide an additional backup feature beyond recovery codes.

Unlike slower blockchains that require Layer 2 solutions for everyday payments, **Kaspa's high throughput and low latency allow all transactions to settle directly on the base layer**. There is no need for offchain payment channels or Lightning-like protocols—every kaspa transaction can be confirmed on-DAG in fractions of a second with minimal fees, making Kaspa viable as money in its native form.

Of note, when receiving funds to a new mobile wallet for the first time, many wallets will often re-verify that you have securely backed-up your recovery code. This can range from a simple prompt to requiring the user to manually re-enter the code.

> **Warning:** Although many legitimate wallets will prompt you to re-enter your recovery code, there are also many malware applications that mimic the design of a wallet, insist you enter your recovery code, and then relay any entered code to the malware developer so they can steal your funds. This is the equivalent of phishing websites that try to trick you into giving them your bank passphrase.
>
> For most wallet applications, the only times they will ask for your recovery code are during the initial set up (before you have received any kaspa) and during recovery (after you lost access to your original wallet). If the application asks for your recovery code any other time, consult with an expert to ensure you aren't being phished.

---

## Kaspa Addresses

Alice is now ready to start using her new Kaspa wallet. Her wallet application randomly generated a private key (described in more detail in later chapters) that will be used to derive Kaspa addresses that direct to her wallet.

At this point, her Kaspa addresses are not known to the Kaspa network or "registered" with any part of the Kaspa system. Her Kaspa addresses are simply numbers that correspond to her private key that she can use to control access to the funds. The addresses are generated independently by her wallet without reference or registration with any service.

> **Note:** There are a variety of Kaspa addresses and invoice formats. Addresses and invoices can be shared with other Kaspa users who can use them to send kaspa directly to your wallet.
>
> You can share an address or invoice with other people without worrying about the security of your kaspa. Unlike a bank account number, nobody who learns one of your Kaspa addresses can withdraw money from your wallet—you must initiate all spends.
>
> However, if you give two people the same address, they will be able to see how many kaspa the other person sent you. If you post your address publicly, everyone will be able to see how much kaspa other people sent to that address.
>
> **To protect your privacy, you should generate a new invoice with a new address each time you request a payment.**

---

## Receiving Kaspa

Alice uses the _Receive_ button, which displays a QR code.

The QR code is the square with a pattern of black and white dots, serving as a form of barcode that contains the same information in a format that can be scanned by Joe's smartphone camera.

> **Warning:** Any funds sent to the addresses in this book will be lost. If you want to test sending kaspa, please consider donating it to a kaspa-accepting charity.

### Getting Your First Kaspa

The first task for new users is to acquire some kaspa.

Kaspa transactions are irreversible. Most electronic payment networks such as credit cards, debit cards, PayPal, and bank account transfers are reversible. For someone selling kaspa, this difference introduces a very high risk that the buyer will reverse the electronic payment after they have received kaspa, in effect defrauding the seller.

To mitigate this risk, companies accepting traditional electronic payments in return for kaspa usually require buyers to undergo identity verification and credit-worthiness checks, which may take several days or weeks. As a new user, this means you cannot buy kaspa instantly with a credit card. With a bit of patience and creative thinking, however, you won't need to.

**Here are some methods for acquiring kaspa as a new user:**

1. **Find a friend who has kaspa** and buy some from him or her directly. Many Kaspa users start this way. This method is the least complicated. One way to meet people with kaspa is to attend a local Kaspa meetup.
2. **Earn kaspa by selling a product or service for kaspa.** If you are a programmer, sell your programming skills. If you're a hairdresser, cut hair for kaspa.
3. **Use a Kaspa ATM in your city.** A Kaspa ATM is a machine that accepts cash and sends kaspa to your smartphone Kaspa wallet.
4. **Use a Kaspa currency exchange linked to your bank account.** Many countries now have currency exchanges that offer a market for buyers and sellers to swap kaspa with local currency. Exchange-rate listing services often show a list of Kaspa exchanges for each currency.

> **Note:** One of the advantages of Kaspa over other payment systems is that, when used correctly, it affords users much more privacy. Acquiring, holding, and spending kaspa does not require you to divulge sensitive and personally identifiable information to third parties.
>
> However, where kaspa touches traditional systems, such as currency exchanges, national and international regulations often apply. In order to exchange kaspa for your national currency, you will often be required to provide proof of identity and banking information.
>
> Users should be aware that once a Kaspa address is attached to an identity, other associated Kaspa transactions may also become easy to identify and track—including transactions made earlier. This is one reason many users choose to maintain dedicated exchange accounts independent from their wallets.

Alice was introduced to Kaspa by a friend, so she has an easy way to acquire her first kaspa. Next, we will look at how she buys kaspa from her friend Joe and how Joe sends the kaspa to her wallet.

### Finding the Current Price of Kaspa

Before Alice can buy kaspa from Joe, they have to agree on the _exchange rate_ between kaspa and US dollars. This brings up a common question for those new to Kaspa: "Who sets the price of kaspa?" The short answer is that **the price is set by markets**.

Kaspa, like most other currencies, has a _floating exchange rate_. That means that the value of kaspa fluctuates according to supply and demand in the various markets where it is traded. For example, the "price" of kaspa in US dollars is calculated in each market based on the most recent trade of kaspa and US dollars.

As such, the price tends to fluctuate minutely several times per second. A pricing service will aggregate the prices from several markets and calculate a volume-weighted average representing the broad market exchange rate of a currency pair (e.g., KAS/USD).

There are many applications and websites that can provide the current market rate.

In addition to these various sites and applications, some Kaspa wallets will automatically convert amounts between kaspa and other currencies.

---

## Sending and Receiving Kaspa

Alice has decided to buy 0.001 kaspa. After she and Joe check the exchange rate, she gives Joe an appropriate amount of cash, opens her mobile wallet application, and selects Receive. This displays a QR code with Alice's first Kaspa address.

Joe then selects Send on his smartphone wallet and opens the QR code scanner. This allows Joe to scan the barcode with his smartphone camera so that he doesn't have to type in Alice's Kaspa address, which is quite long.

Joe now has Alice's Kaspa address set as the recipient. Joe enters the amount as **0.001 kaspa**. Some wallets may show the amount in a different denomination: 0.001 kaspa is **1 millikaspa (mKAS)** or **1,000,000 sompi**.

Some wallets may also suggest Joe enter a label for this transaction; if so, Joe enters "Alice". Weeks or months from now, this will help Joe remember why he sent these 0.001 kaspa.

Some wallets may also prompt Joe about fees. Depending on the wallet and how the transaction is being sent, the wallet may ask Joe to either enter a transaction fee rate or prompt him with a suggested fee (or fee rate). The higher the transaction fee, the faster the transaction will be confirmed.

Joe then carefully checks to make sure he has entered the correct amount, because he is about to transmit money and mistakes will soon become irreversible. After double-checking the address and amount, he presses Send to transmit the transaction.

Joe's mobile Kaspa wallet constructs a transaction that assigns 0.001 kaspa to the address provided by Alice, sourcing the funds from Joe's wallet, and signing the transaction with Joe's private keys. This tells the Kaspa network that Joe has authorized a transfer of value to Alice's new address.

As the transaction is transmitted via the peer-to-peer protocol, it quickly propagates across the Kaspa network. After just a few seconds, most of the well-connected nodes in the network receive the transaction and see Alice's address for the first time.

Meanwhile, Alice's wallet is constantly "listening" for new transactions on the Kaspa network, looking for any that match the addresses it contains. A few seconds after Joe's wallet transmits the transaction, Alice's wallet will indicate that it is receiving **0.001 kaspa**.

### Confirmations

At first, Alice's address will show the transaction from Joe as "Unconfirmed." This means that the transaction has been propagated to the network but has not yet been recorded in the Kaspa transaction journal, known as the blockDAG.

To be confirmed, a transaction must be included in a block and added to the blockDAG, which happens **every 0.1 seconds (100ms), on average**. In traditional financial terms this is known as _clearing_. For more details on propagation, validation, and clearing (confirmation) of kaspa transactions, see the mining chapter.

---

Alice is now the proud owner of **0.001 kaspa** that she can spend. Over the next few days, Alice buys more kaspa using an ATM and an exchange. In the next chapter we will look at her first purchase with Kaspa, and examine the underlying transaction and propagation technologies in more detail.
