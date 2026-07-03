# How Kaspa Works

The Kaspa system, unlike traditional banking and payment systems, does not require trust in third parties. Instead of a central trusted authority, in Kaspa, each user can use software running on their own computer to verify the correct operation of every aspect of the Kaspa system.
In this chapter, we will examine Kaspa from a high level by tracking a single transaction through the Kaspa system and watch as it is recorded on the blockDAG, the distributed journal of all transactions. Subsequent chapters will delve into the technology behind transactions, the network, and mining.

## Kaspa Overview

The Kaspa system consists of users with wallets containing keys, transactions that are propagated across the network, and miners who produce (through competitive computation) the consensus blockDAG, which is the authoritative journal of all transactions.

Each example in this chapter is based on an actual transaction made on the Kaspa network, simulating the interactions between several users by sending funds from one wallet to another. While tracking a transaction through the Kaspa network to the blockDAG, we will use a *blockDAG explorer* site to visualize each step. A blockDAG explorer is a web application that operates as a Kaspa search engine, in that it allows you to search for addresses, transactions, and blocks and see the relationships and flows between them.

Popular blockDAG explorers include the following:

- https://explorer.kaspa.org [Official Kaspa Explorer]
- https://kaspa.stream [Kaspa Stream]


Each of these has a search function that can take a Kaspa address, transaction hash, block number, or block hash and retrieve corresponding information from the Kaspa network. With each transaction or block example, we will provide a URL so you can look it up yourself and study it in detail.

> **Warning:** Block Explorer Privacy Warning
>
> Searching information on a blockDAG explorer may disclose to its operator that you're interested in that information, allowing them to associate it with your IP address, browser details, past searches, or other identifiable information. If you look up the transactions in this book, the operator of the blockDAG explorer might guess that you're learning about Kaspa, which shouldn't be a problem. But if you look up your own transactions, the operator may be able to guess how many kaspa you've received, spent, and currently own.

---

## Buying from an Online Store

Alice, introduced in the previous chapter, is a new user who has just acquired her first kaspa. In the earlier section, Alice met with her friend Joe to exchange some cash for kaspa. Since then, Alice has bought additional kaspa. Now Alice will make her first spending transaction, buying access to a premium podcast episode from Bob's online store.

Bob's web store recently started accepting Kaspa payments by adding a Kaspa option to its website. The prices at Bob's store are listed in the local currency (US dollars), but at checkout, customers have the option of paying in either dollars or kaspa.

Alice finds the podcast episode she wants to buy and proceeds to the checkout page. At checkout, Alice is offered the option to pay with Kaspa in addition to the usual options. The checkout cart displays the price in US dollars and also in kaspa (KAS), at Kaspa's prevailing exchange rate.

Bob's ecommerce system will automatically create a QR code containing an *invoice*.

Unlike a QR code that simply contains a destination Kaspa address, this invoice is a QR-encoded URI that contains a destination address, a payment amount, and a description. This allows a Kaspa wallet application to prefill the information used to send the payment while showing a human-readable description to the user. You can scan the QR code with a kaspa wallet application to see what Alice would see:

TODO: Add qr code for sample

The invoice QR code encodes the following URI:

```
kaspa:address_here?amount=0.5&label=Bob%27s%20Store&message=Purchase%20at%20Bob%27s%20Store
```

**Components of the URI:**

- A Kaspa address: "address_here"
- The payment amount: "0.5"
- A label for the recipient address: "Bob's Store"
- A description for the payment: "Purchase at Bob's Store"

> **Tip:** Try to scan this with your wallet to see the address and amount but DO NOT SEND MONEY.

Alice uses her smartphone to scan the barcode on display. Her smartphone shows a payment for the correct amount to *Bob's Store* and she selects Send to authorize the payment. Within a second (faster than a credit card authorization), Bob sees the transaction on the register.

> **Note:** The Kaspa network can transact in fractional values, e.g., from millikaspa (1/1000th of a kaspa) down to 1/100,000,000th of a kaspa, which is known as a sompi. This book uses the same pluralization rules used for dollars and other traditional currencies when talking about amounts greater than one kaspa and when using decimal notation, such as "10 kaspa" or "0.001 kaspa." The same rules also apply to other Kaspa bookkeeping units, such as millikaspa and sompi.

You can use a blockDAG explorer to examine blockDAG data, such as the payment made to Bob in Alice's transaction.

In the following sections, we will examine this transaction in more detail. We'll see how Alice's wallet constructed it, how it was propagated across the network, how it was verified, and finally, how Bob can spend that amount in subsequent transactions.

---

## Kaspa Transactions

In simple terms, a transaction tells the network that the owner of certain kaspa has authorized the transfer of that value to another owner. The new owner can now spend the kaspa by creating another transaction that authorizes the transfer to another owner, and so on, in a chain of ownership.

### Transaction Inputs and Outputs

Transactions are like lines in a double-entry bookkeeping ledger. Each transaction contains one or more *inputs*, which spend funds. On the other side of the transaction, there are one or more *outputs*, which receive funds. The inputs and outputs do not necessarily add up to the same amount. Instead, outputs add up to slightly less than inputs and the difference represents an implied _transaction fee_, which is a small payment collected by the miner who includes the transaction in the blockDAG. A Kaspa transaction is shown as a bookkeeping ledger entry.

The transaction also contains proof of ownership for each amount of kaspa (inputs) whose value is being spent, in the form of a digital signature from the owner, which can be independently validated by anyone. In Kaspa terms, spending is signing a transaction that transfers value from a previous transaction over to a new owner identified by a Kaspa address.

### Transaction Chains

Alice's payment to Bob's Store uses a previous transaction's output as its input. In the previous chapter, Alice received kaspa from her friend Joe in return for cash. We've labeled that as _Transaction 1_ (Tx1).

Tx1 sent 0.001 kaspa (100,000 sompi) to an output locked by Alice's key. Her new transaction to Bob's Store (Tx2) references the previous output as an input. In the illustration, we show that reference using an arrow and by labeling the input as "Tx1:0". In an actual transaction, the reference is the 32-byte transaction identifier (txid) for the transaction where Alice received the money from Joe. The ":0" indicates the position of the output where Alice received the money; in this case, the first position (position 0).

As shown, actual Kaspa transactions don't explicitly include the value of their input. To determine the value of an input, software needs to use the input's reference to find the previous transaction output being spent.

Alice's Tx2 contains two new outputs, one paying 75,000 sompi for the podcast and another paying 20,000 sompi back to Alice to receive change.

> **Tip:** Serialized Kaspa transactions—the data format that software uses for sending transactions—encodes the value to transfer using an integer of the smallest defined on-DAG unit of value. When Kaspa was first created, users began calling this unit a _sompi_ in honor of the protocol's precision, later it was sugggested to rename this unit to dwork. In illustrations throughout this book, we use sompi values because that's what the protocol itself uses.

### Making Change

In addition to one or more outputs that pay the receiver of kaspa, many transactions will also include an output that pays the spender of the kaspa, called a _change_ output. This is because transaction inputs, like currency notes, cannot be partly spent. If you purchase a $5 US item in a store but use a $20 bill to pay for the item, you expect to receive $15 in change. The same concept applies to Kaspa transaction inputs. If you purchased an item that costs 5 kaspa but only had an input worth 20 kaspa to use, you would send one output of 5 kaspa to the store owner and one output of 15 kaspa back to yourself as change (not counting your transaction fee).

At the level of the Kaspa protocol, there is no difference between a change output (and the address it pays, called a _change address_) and a payment output.

Importantly, the change address does not have to be the same address as that of the input and, for privacy reasons, is often a new address from the owner's wallet. In ideal circumstances, the two different uses of outputs both use never-before-seen addresses and otherwise look identical, preventing any third party from determining which outputs are change and which are payments. However, for illustration purposes, we've added shading to the change outputs.

Not every transaction has a change output. Those that don't are called _changeless transactions_, and they can have only a single output. Changeless transactions are only a practical option if the amount being spent is roughly the same as the amount available in the transaction inputs minus the anticipated transaction fee. In our example, we see Bob creating Tx3 as a changeless transaction that spends the output he received in Tx2.

### Coin Selection

Different wallets use different strategies when choosing which inputs to use in a payment, called _coin selection_. They might aggregate many small inputs, or use one that is equal to or larger than the desired payment. Unless the wallet can aggregate inputs in such a way to exactly match the desired payment plus transaction fees, the wallet will need to generate some change. This is very similar to how people handle cash. If you always use the largest bill in your pocket, you will end up with a pocket full of loose change. If you only use the loose change, you'll often have only big bills. People subconsciously find a balance between these two extremes, and Kaspa wallet developers strive to program this balance.

### Common Transaction Forms

A very common form of transaction is a simple payment. This type of transaction has one input and two outputs.

Another common form of transaction is a _consolidation transaction_, which spends several inputs into a single output. This represents the real-world equivalent of exchanging a pile of coins and currency notes for a single larger note. Transactions like these are sometimes generated by wallets and businesses to clean up lots of smaller amounts.

Finally, another transaction form that is seen often on the blockDAG is _payment batching_, which pays to multiple outputs representing multiple recipients. This type of transaction is sometimes used by commercial entities to distribute funds, such as when processing payroll payments to multiple employees.

---

## Constructing a Transaction

Alice's wallet application contains all the logic for selecting inputs and generating outputs to build a transaction to Alice's specification. Alice only needs to choose a destination, amount, and transaction fee, and the rest happens in the wallet application without her seeing the details. Importantly, if a wallet already knows what inputs it controls, it can construct transactions even if it is completely offline.
Like writing a check at home and later sending it to the bank in an envelope, the transaction does not need to be constructed and signed while connected to the Kaspa network.

### Getting the Right Inputs

Alice's wallet application will first have to find inputs that can pay the amount she wants to send to Bob. Most wallets keep track of all the available outputs belonging to addresses in the wallet. Therefore, Alice's wallet would contain a copy of the transaction output from Joe's transaction, which was created in exchange for cash.
A Kaspa wallet application that runs on a full node actually contains a copy of every confirmed transaction's unspent outputs, called _unspent transaction outputs_ (UTXOs). However, because full nodes use more resources, many user wallets run lightweight clients that track only the user's own UTXOs.

In this case, this single UTXO is sufficient to pay for the podcast. Had this not been the case, Alice's wallet application might have to combine several smaller UTXOs, like picking coins from a purse, until it could find enough to pay for the podcast. In both cases, there might be a need to get some change back, which we will see in the next section, as the wallet application creates the transaction outputs (payments).

### Creating the Outputs

A transaction output is created with a script that says something like, "This output is paid to whoever can present a signature from the key corresponding to Bob's public address." Because only Bob has the wallet with the keys corresponding to that address, only Bob's wallet can present such a signature to later spend this output. Alice will therefore _encumber_ the output value with a demand for a signature from Bob.

This transaction will also include a second output because Alice's funds contain more money than the cost of the podcast. Alice's change output is created in the very same transaction as the payment to Bob. Essentially, Alice's wallet breaks her funds into two outputs: one to Bob and one back to herself. She can then spend the change output in a subsequent transaction.

Finally, for the transaction to be processed by the network in a timely fashion, Alice's wallet application will add a small fee. The fee is not explicitly stated in the transaction; it is implied by the difference in value between inputs and outputs. This transaction fee is collected by the miner as a fee for including the transaction in a block that gets recorded on the blockDAG.

### Adding the Transaction to the BlockDAG

The transaction created by Alice's wallet application contains everything necessary to confirm ownership of the funds and assign new owners. Now, the transaction must be transmitted to the Kaspa network where it will become part of the blockDAG. In the next section we will see how a transaction becomes part of a new block and how the block is mined. Finally, we will see how the new block, once added to the blockDAG, is increasingly trusted by the network as more blocks are added.

#### Transmitting the transaction

Because the transaction contains all the information necessary for it to be processed, it does not matter how or where it is transmitted to the Kaspa network. The Kaspa network is a peer-to-peer network, with each Kaspa peer participating by connecting to several other Kaspa peers. The purpose of the Kaspa network is to propagate transactions and blocks to all participants.

#### How it propagates

Peers in the Kaspa peer-to-peer network are programs that have both the software logic and the data necessary for them to fully verify the correctness of a new transaction. The connections between peers are often visualized as edges (lines) in a graph, with the peers themselves being the nodes (dots). For that reason, Kaspa peers are commonly called "full verification nodes," or _full nodes_ for short.

Alice's wallet application can send the new transaction to any Kaspa node over any type of connection: wired, WiFi, mobile, etc. It can also send the transaction to another program (such as a blockDAG explorer) that will relay it to a node. Her Kaspa wallet does not have to be connected to Bob's Kaspa wallet directly and she does not have to use the internet connection offered by Bob, though both those options are possible too. Any Kaspa node that receives a valid transaction it has not seen before will forward it to all other nodes to which it is connected, a propagation technique known as _gossiping_. Thus, the transaction rapidly propagates out across the peer-to-peer network, reaching a large percentage of the nodes within a few seconds.

#### Bob's view

If Bob's Kaspa wallet application is directly connected to Alice's wallet application, Bob's wallet application might be the first to receive the transaction. However, even if Alice's wallet sends the transaction through other nodes, it will reach Bob's wallet within a few seconds. Bob's wallet will immediately identify Alice's transaction as an incoming payment because it contains an output redeemable by Bob's keys. Bob's wallet application can also independently verify that the transaction is well formed. If Bob is using his own full node, his wallet can further verify Alice's transaction only spends valid UTXOs.

---

## Kaspa Mining

Alice's transaction is now propagated on the Kaspa network. It does not become part of the _blockDAG_ until it is included in a block by a process called _mining_ and that block has been validated by full nodes.

Kaspa's system of counterfeit protection is based on computation. Transactions are bundled into _blocks_. Blocks have a very small header that must be formed in a very specific way, requiring an enormous amount of computation to get right—but only a small amount of computation to verify as correct.
The mining process serves two purposes in Kaspa:

* Miners can only receive honest income from creating blocks that follow all of Kaspa's _consensus rules_. Therefore, miners are normally incentivized to only include valid transactions in their blocks and the blocks they build upon. This allows users to optionally make a trust-based assumption that any transaction in a block is a valid transaction.

* Mining currently creates new kaspa in each block, almost like a central bank printing new money. The amount of kaspa created per block is limited and diminishes with time, following a fixed issuance schedule.

Mining achieves a fine balance between cost and reward. Mining uses electricity to solve a computational problem. A successful miner will collect a _reward_ in the form of new kaspa and transaction fees. However, the reward will only be collected if the miner has only included valid transactions, with the Kaspa protocol's rules for _consensus_ determining what is valid. This delicate balance provides security for Kaspa without a central authority.

Mining is designed to be a decentralized lottery. Each miner can create their own lottery ticket by creating a _candidate block_ that includes the new transactions they want to mine plus some additional data fields. The miner inputs their candidate into a specially designed algorithm that scrambles (or "hashes") the data, producing output that looks nothing like the input data. This _hash_ function will always produce the same output for the same input—but nobody can predict what the output will look like for a new input, even if it is only slightly different from a previous input. If the output of the hash function matches a template determined by the Kaspa protocol, the miner wins the lottery and Kaspa users will accept the block with its transactions as a valid block. If the output doesn't match the template, the miner makes a small change to their candidate block and tries again.

However, once a winning combination has been found, anyone can verify the block is valid by running the hash function just once. That makes a valid block something that requires an incredible amount of work to create but only a trivial amount of work to verify. The simple verification process is able to probabilistically prove the work was done, so the data necessary to generate that proof—in this case, the block—is called _proof of work (PoW)_.

Transactions are added to the new block, prioritized by the highest fee rate transactions first and a few other criteria. Each miner starts the process of mining a new candidate block of transactions as soon as they receive the previous block from the network, knowing that some other miner won that iteration of the lottery. They immediately create a new candidate block with a commitment to the previous block, fill it with transactions, and start calculating the PoW for the candidate block. Each miner includes a special transaction in their candidate blocks, one that pays their own Kaspa address the block reward plus the sum of transaction fees from all the transactions included in the candidate block. If they find a solution that makes the candidate into a valid block, they receive this reward after their successful block is added to the global blockDAG and the reward transaction they included becomes spendable. Miners who participate in a mining pool have set up their software to create candidate blocks that assign the reward to a pool address. From there, a share of the reward is distributed to members of the pool miners in proportion to the amount of work they contributed.

Alice's transaction was picked up by the network and included in the pool of unverified transactions. Once validated by a full node, it was included in a candidate block. Approximately a few seconds after the transaction was first transmitted by Alice's wallet, a miner finds a solution for the block and announces it to the network. After each other miner validates the winning block, they start a new lottery to generate the next block.

The winning block containing Alice's transaction became part of the blockDAG. The block containing Alice's transaction is counted as one _confirmation_ of that transaction. After the block containing Alice's transaction has propagated through the network, creating an alternative block with a different version of Alice's transaction (such as a transaction that doesn't pay Bob) would require performing the same amount of work as it will take all Kaspa miners to create an entirely new block. When there are multiple alternative blocks to choose from, Kaspa full nodes use the GHOSTDAG protocol to determine the canonical ordering of blocks in the blockDAG. For the entire network to accept an alternative block, an additional new block would need to be mined on top of the alternative.

That means miners have a choice. They can work with Alice on an alternative to the transaction where she pays Bob, perhaps with Alice paying miners a share of the money she previously paid Bob. This dishonest behavior will require they expend the effort required to create two new blocks. Instead, miners who behave honestly can create a single new block and receive all of the fees from the transactions they include in it, plus the block subsidy. Normally, the high cost of dishonestly creating two blocks for a small additional payment is much less profitable than honestly creating a new block, making it unlikely that a confirmed transaction will be deliberately changed. For Bob, this means that he can begin to believe that the payment from Alice can be relied upon.

> **Tip:** You can see the block that includes Alice's transaction on any Kaspa blockDAG explorer.

Because Kaspa produces blocks every 0.1 seconds (100 milliseconds), confirmations accumulate rapidly. After just a few seconds, multiple blocks are mined on top of the one containing Alice's transaction, giving it several confirmations. Each block mined on top of the one containing Alice's transaction counts as an additional confirmation. As the blocks pile on top of each other, it becomes harder to reverse the transaction, thereby giving Bob more and more confidence that Alice's payment is secure.

In Kaspa's blockDAG structure, blocks can be created concurrently by different miners, forming a directed acyclic graph rather than a simple linear chain. The GHOSTDAG protocol provides a deterministic way to order these blocks and achieve consensus on the canonical transaction history. Over time, as the "height" of new blocks increases, so does the computation difficulty for the network as a whole.

---

## Spending the Transaction

Now that Alice's transaction has been embedded in the blockDAG as part of a block, it is visible to all Kaspa applications. Each Kaspa full node can independently verify the transaction as valid and spendable. Full nodes validate every transfer of the funds from the moment the kaspa were first generated in a block through each subsequent transaction until they reach Bob's address, this validation happens through maintaining the UTXO set, only spendable UTXO's enter the UTXO set, are committed cryptographically to each header, and are removed as they are spent, this all happens in consensus for every block, the UTXO set MUST be valid and correct for a block to be valid. Lightweight clients can partially verify payments by confirming that the transaction is in the blockDAG and has several blocks mined after it, thus providing assurance that the miners expended significant effort committing to it.

Bob can now spend the output from this and other transactions. For example, Bob can pay a contractor or supplier by transferring value from Alice's podcast payment to these new owners. As Bob spends the payments received from Alice and other customers, he extends the chain of transactions. Let's assume that Bob pays his web designer Gopesh for a new website page. Now the chain of transactions will extend further.

In this chapter, we saw how transactions build a chain that moves value from owner to owner. We also tracked Alice's transaction from the moment it was created in her wallet, through the Kaspa network, and to the miners who recorded it on the blockDAG. In the rest of this book, we will examine the specific technologies behind wallets, addresses, signatures, transactions, the network, and finally, mining.
