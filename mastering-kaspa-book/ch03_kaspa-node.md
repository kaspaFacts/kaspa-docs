# Kaspa Node: The Reference Implementation

People only accept money in exchange for their valuable goods and services if they believe that they'll be able to spend that money later. Money that is counterfeit or unexpectedly debased may not be spendable later, so every person accepting kaspa has a strong incentive to verify the integrity of the kaspa they receive. The Kaspa system was designed so that it's possible for software running entirely on your local computer to perfectly prevent counterfeiting, debasement, and several other critical problems. Software which provides that function is called a _full verification node_ because it verifies every confirmed Kaspa transaction against every rule in the system. Full verification nodes, _full nodes_ for short, may also provide tools and data for understanding how Kaspa works and what is currently happening in the network.

In this chapter, we'll install kaspad, the reference implementation that most full node operators use on the Kaspa network. We'll then inspect blocks, transactions, and other data from your node—data which is authoritative not because some powerful entity designated it as such but because your node independently verified it. Throughout the rest of this book, we'll continue using kaspad to create and examine data related to the blockDAG and network.

**Note:** This chapter provides an introduction to running a Kaspa node and interacting with the network. However, the Kaspa protocol and `kaspad` implementation are actively developed, and configurations, RPC methods, and network parameters may change between releases. For the most current and authoritative information, always refer to the official documentation in the `rusty-kaspa` repository and the latest release notes. The `stable` branch contains code corresponding to the latest stable release, while the `master` branch receives ongoing development.

## From Kaspa to kaspad

Kaspa is an _open source_ project written in **Rust**, and the source code is available under an open license, free to download and use for any purpose. More than just being open source, Kaspa is developed by an open community of volunteers. The Kaspa protocol was introduced in 2021 with the PHANTOM GHOSTDAG paper by Yonatan Sompolinsky, Shai Wyborski, and Aviv Zohar from The Hebrew University of Jerusalem. Since then, Kaspa's source code has had numerous contributors working on the code full time and part-time. Anyone can contribute to the code—including you!

The first implementation, simply known as "Kaspa," has been heavily modified and improved since its launch in 2021. It has evolved into what is known as _kaspad_, to differentiate it from other implementations. kaspad is the _reference implementation_ of the Kaspa system, meaning that it provides a reference for how each part of the technology should be implemented. kaspad implements all aspects of Kaspa, including wallets, a transaction and block validation engine, tools for block construction, and all modern parts of Kaspa peer-to-peer communication.

Although kaspad serves as a reference implementation for many major parts of the system, the PHANTOM GHOSTDAG paper describes several early parts of the system. Throughout this book, we refer to specifications by their documentation in the Kaspa repository.

## Kaspa Development Environment

If you're a developer, you will want to set up a development environment with all the tools, libraries, and support software for writing Kaspa applications. In this highly technical chapter, we'll walk through that process step by step. If the material becomes too dense (and you're not actually setting up a development environment) feel free to skip to the next chapter, which is less technical.

## Compiling kaspad from the Source Code

rusty-kaspa's source code can be downloaded as an archive or by cloning the source repository from GitHub. On the Kaspa GitHub page, select the most recent version and download the compressed archive of the source code. Alternatively, use the Git command line to create a local copy of the source code:

```bash
$ git clone https://github.com/kaspanet/rusty-kaspa.git
Cloning into 'rusty-kaspa'...
remote: Enumerating objects...
Receiving objects: 100%...
Resolving deltas: 100%...
done.
```

> **Tip:** In many of the examples in this chapter, we will be using the operating system's command-line interface (also known as a "shell"), accessed via a "terminal" application. The shell will display a prompt, you type a command, and the shell responds with some text and a new prompt for your next command. The prompt may look different on your system, but in the following examples, it is denoted by a `$` symbol. In the examples, when you see text after a `$` symbol, don't type the `$` symbol but type the command immediately following it, then press Enter to execute the command.

> **Tip:** Git is the most widely used distributed version control system, an essential part of any software developer's toolkit. You may need to install the `git` command, or a graphical user interface for Git, on your operating system if you do not have it already.

When the Git cloning operation has completed, you will have a complete local copy of the source code repository in the directory _rusty-kaspa_. Change to this directory using the `cd` command:

```bash
$ cd rusty-kaspa
```

### Selecting a kaspad Release

By default, the local copy will be synchronized with the most recent code, which might be an unstable or beta version of Kaspa. Before compiling the code, select a specific version by checking out a release _tag_. This will synchronize the local copy with a specific snapshot of the code repository identified by a keyword tag. Tags are used by the developers to mark specific releases of the code by version number.

First, to find the available tags, we use the `git tag` command:

```bash
$ git tag
v1.0.0
v1.0.1
...
v1.10.0
v1.10.1rc1
...
```

The list of tags shows all the released versions of Kaspa. By convention, _release candidates_, which are intended for testing, have the suffix "rc." Stable releases that can be run on production systems have no suffix.

To synchronize the local code with a specific version, use the `git checkout` command:

```bash
$ git checkout v1.10.0
Note: switching to 'v1.10.0'.

You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can discard any commits you make in this state without impacting any branches by switching back to a branch.

HEAD is now at [commit] Merge release v1.10.0
```

You can confirm you have the desired version "checked out" by issuing the command `git status`:

```bash
$ git status
HEAD detached at v1.10.0
nothing to commit, working tree clean
```

### Building kaspad

kaspad is written in **Rust**, which provides memory safety and high performance. The source code includes documentation, which can be found in the README.md file in the kaspad directory.

First, ensure you have Rust installed (rustup is recommended):

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

To build kaspad for production use:

```bash
$ cargo build --release --bin kaspad
```

This compiles the kaspad daemon with optimizations enabled. The binary will be located at:

```bash
target/release/kaspad
```

You can verify the build by checking the version:

```bash
$ ./target/release/kaspad --version
kaspad v[version]
```

---

## Running a Kaspa Node

Kaspa's peer-to-peer network is composed of network "nodes," run mostly by individuals and some of the businesses that provide Kaspa services. Those running Kaspa nodes have a direct and authoritative view of the Kaspa blockDAG, with a local copy of all the spendable kaspa independently validated by their own system. By running a node, you don't have to rely on any third party to validate a transaction. Additionally, by using a Kaspa node to fully validate the transactions you receive to your wallet, you contribute to the Kaspa network and help make it more robust.

Running a node requires downloading and processing significant amounts of data initially. As of 2026 (Toccata era), the Kaspa blockDAG is continuously growing with blocks produced every 0.1 seconds (10 blocks per second).

### Hardware Requirements

#### Minimum Specifications

| Component | Requirement |
|-----------|-------------|
| CPU | 8 cores |
| RAM | 16 GB |
| Storage | 640 GB SSD |
| Network | 10 MB/s (~80 Mbit/s) bandwidth |

#### Preferred for Higher Performance

| Component | Recommendation |
|-----------|---------------|
| CPU | 12–16 cores |
| RAM | 32 GB |
| Storage | 1 TB SSD |
| Network | Higher bandwidth for robust peer support |

**Why upgrade?** While minimum specs suffice to sync and maintain a node, increasing resources allows your node to serve as a stronger focal point on the network. This leads to faster initial block download (IBD) for peers syncing from your node and provides more leeway for future storage growth.

> **Note:** If you are a pool operator, it is strongly recommended that you use specifications closer to the preferred recommendations above.

### Why These Requirements Changed

The hardware requirements have been updated for two reasons:

1. **More accurate measurements** from network analysis
2. **Doubling of the transient mass limit** to allow ZK-STARK proofs

### Storage Model: Pruned by Default

**Important:** Unlike Bitcoin, **kaspad full nodes are pruned by default**. This means they do NOT keep a complete copy of the entire blockDAG forever—storage requirements are automatically bounded.

| Node Type | Minimum Storage | Recommended |
|-----------|-----------------|-------------|
| Pruned Full Node (default) | 640 GB SSD | 1 TB SSD for higher performance |
| Archival Node | Varies by retention period | Depends on usage; plan for significant storage |

**Pruned nodes** automatically delete old block data beyond a configurable retention period, while still maintaining full validation capabilities. This is the default and recommended setup for most users.

**Archival nodes** preserve all historical data forever using the `--archival` flag. These are useful for block explorers, research, or if you need to query very old transactions—but they are **NOT required for network operation**.

If you shut down your node or get disconnected from the internet for several days, your node will need to download the data that it missed when you restart. Kaspa produces approximately 864,000 blocks per day (10 blocks/second × 86,400 seconds).

By default, Kaspa nodes also transmit transactions and blocks to other nodes (called "peers"), consuming upload internet bandwidth. If your internet connection is limited, has a low data cap, or is metered (charged by the gigabyte), you should probably not run a Kaspa node on it, or run it in a way that constrains its bandwidth.

> **Tip:** kaspad keeps a pruned copy of the blockDAG by default, bounding storage requirements to hundreds of gigabytes rather than terabytes.

Despite these resource requirements, thousands of people run Kaspa nodes.

Why would you want to run a node? Here are some of the most common reasons:

* You do not want to rely on any third party to validate the transactions you receive.
* You do not want to disclose to third parties which transactions belong to your wallet.
* You are developing Kaspa software and need to rely on a Kaspa node for programmable (API) access to the network and blockDAG.
* You are building applications that must validate transactions according to Kaspa's consensus rules. Typically, Kaspa software companies run several nodes.
* You want to support Kaspa. Running a node that you use to validate the transactions you receive to your wallet makes the network more robust.

If you're reading this book and interested in strong security, superior privacy, or developing Kaspa software, you should be running your own node.

---

## Configuring the Kaspa Node

kaspad will look for a configuration file in its data directory on every start. In this section we will examine the various configuration options and set up a configuration file.

To locate the configuration file, run `kaspad` in your terminal:

```bash
$ kaspad
2024-01-15T10:21:42Z INFO rusty-kaspa version v[version]
2024-01-15T10:21:42Z INFO Using data directory /home/user/.rusty-kaspa
2024-01-15T10:21:42Z INFO Config file: /home/user/.rusty-kaspa/kaspad.conf
...
```

Hit Ctrl-C to shut down the node once you determine the location of the config file.

The default data directories are:

| OS | Data Directory |
|----|---------------|
| Linux | `~/.rusty-kaspa/` |
| macOS | `~/.rusty-kaspa/` |
| Windows | `%LOCALAPPDATA%\rusty-kaspa\` |

The configuration file is located at `<data_directory>/kaspad.conf`.

Open the configuration file in your preferred editor. kaspad uses a **TOML-formatted** configuration file called `kaspad.conf`.

kaspad offers many configuration options that modify the behavior of the network node, the storage of the blockDAG, and many other aspects of its operation. To see available command-line options:

```bash
$ kaspad --help
```

Here are some of the most important options:

### Network Selection

Use boolean flags to select the network (mainnet is default when no flag is set):

**Command-line:**
```bash
$ kaspad --testnet      # Run on testnet
$ kaspad --devnet       # Run on devnet
$ kaspad --simnet       # Run on simnet
$ kaspad                # Runs on mainnet (default)
```

**TOML config:**
```toml
testnet = true    # Set to true for testnet, false or omit for mainnet
devnet = false    # Set to true for devnet
simnet = false    # Set to true for simnet
```

Kaspa supports multiple networks:
- **mainnet**: The production Kaspa network with real kaspa (default)
- **testnet**: A testing network with no-value test kaspa
- **devnet**: A development network for local testing
- **simnet**: A simulation network for testing consensus algorithms

### Default Ports

| Service | Mainnet | Testnet |
|---------|---------|----------|
| JSON-RPC | 16110 | 16210 |
| P2P Network | 16111 | 16211 |

Configure RPC listen address (command-line uses `--rpclisten`):

```toml
# TOML config option name may vary; verify with kaspad --help
rpc_listen_address = "127.0.0.1:16110"
```

### Storage and Pruning Configuration

**Pruned node (default, recommended):**

```bash
$ kaspad --retention-period-days 3
```

This keeps only the last 3 days of block data while maintaining full validation. Adjust based on your storage capacity; not adjusting will result in using the default retention period of about 30 hours.

For archival nodes that keep all historical data, use the `--archival` flag (see Sample Command-Line Configurations below).

### Memory and Performance

kaspad uses the `--ram-scale` option to control memory allocation:

```bash
$ kaspad --ram-scale 1.0  # Default, suitable for ~16-32GB RAM systems
```

Recommended values:
- **0.5**: Systems with minimum 16 GB RAM
- **1.0** (default): Systems with 16–32 GB RAM
- **3.0–4.0**: Systems with 64 GB RAM
- Range: 0.1 to 10.0

### Mining Note

kaspad itself does not mine blocks directly. For mining, use external miners like **kaspa-miner**. kaspad provides these flags for special cases:

- `--enable-unsynced-mining`: Accept blocks from RPC while not fully synced (testing/devnet)
- `--enable-mainnet-mining`: Currently enabled by default for backwards compatibility

### Sample Command-Line Configurations

**Non-Standard pruned full node (custom configuring a retention period):**

```bash
$ kaspad --retention-period-days 30 --ram-scale 1.0
```

**Archival node** (keeps all historical data):

```bash
$ kaspad --archival --ram-scale 3.0
```

**Testnet node:**

```bash
$ kaspad --testnet
```

**Devnet node:**

```bash
$ kaspad --devnet
```

---

## Kaspa Node API

kaspad implements multiple RPC interfaces for programmatic access:

- **wRPC** (WebSocket-based RPC) - Primary interface using WebSocket protocol with Borsh or JSON encoding
- **gRPC** - Another protocol option for high-performance communication

The examples below use HTTP JSON-RPC format for simplicity, but production applications typically use wRPC over WebSockets for better performance and real-time capabilities.

### Using curl for RPC Calls

The simplest way to make RPC calls is using `curl`, the versatile command-line HTTP client. Note that this uses HTTP JSON-RPC format for demonstration; production applications typically use wRPC over WebSockets.

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetServerInfo","id":1}'
```

This submits a JSON-RPC request to the local kaspad instance on port **16110** (mainnet).

### Common RPC Methods

Here are some commonly used kaspad RPC methods:

#### GetServerInfo

Returns general information about the node.

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetServerInfo","id":1}'
```

Response:
```json
{
  "result": {
    "rpc_api_version": 1,
    "rpc_api_revision": 0,
    "server_version": "x.x.x",
    "network_id": "kaspa-mainnet",
    "has_utxo_index": true,
    "is_synced": false,
    "virtual_daa_score": 123456
  },
  "error": null,
  "id": 1
}
```

#### GetSyncStatus

Returns the synchronization status:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetSyncStatus","id":1}'
```

Response:
```json
{
  "result": {
    "is_synced": false
  },
  "error": null,
  "id": 1
}
```

#### GetBlockCount

Returns the number of blocks in the blockDAG:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetBlockCount","id":1}'
```

Response:
```json
{
  "result": {
    "block_count": 1000000,
    "header_count": 1000000
  },
  "error": null,
  "id": 1
}
```

#### GetBlock

Returns detailed information about a specific block by hash:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetBlock","params":["block_hash_here"],"id":1}'
```

#### GetTransaction

Returns information about a specific transaction:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetTransaction","params":["tx_hash_here"],"id":1}'
```

#### GetConnectedPeerInfo

Returns information about connected peers:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetConnectedPeerInfo","id":1}'
```

### Kaspa-Specific BlockDAG RPC Methods

Kaspa provides several unique RPC methods for blockDAG operations:

#### GetBlockDagInfo

Returns comprehensive information about the blockDAG structure:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetBlockDagInfo","id":1}'
```

Response:
```json
{
  "result": {
    "network_name": "kaspa-mainnet",
    "block_count": 1000000,
    "header_count": 1000000,
    "tip_hashes": ["hash1", "hash2"],
    "difficulty": 12345.67,
    "past_median_time": 1234567890,
    "virtual_parent_hashes": ["hash3"],
    "pruning_point_hash": "hash4",
    "virtual_daa_score": 123456,
    "sink": "hash5"
  },
  "error": null,
  "id": 1
}
```

#### GetSink

Returns the highest cumulative difficulty block (the "tip" of the main chain):

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetSink","id":1}'
```

Response:
```json
{
  "result": {
    "sink": "block_hash_here"
  },
  "error": null,
  "id": 1
}
```

#### GetSinkBlueScore

Returns the total work on the main chain (blue score):

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetSinkBlueScore","id":1}'
```

Response:
```json
{
  "result": {
    "blue_score": 1000000
  },
  "error": null,
  "id": 1
}
```

#### GetCurrentBlockColor

Checks if a block is blue (part of GHOSTDAG consensus):

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetCurrentBlockColor","params":["block_hash"],"id":1}'
```
```

#### GetBlockRewardInfo

Returns reward information for pool accounting:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetBlockRewardInfo","id":1}'
```

Response:
```json
{
  "result": {
    "block_color": "BLUE",
    "confirmation_count": 100,
    "reward": 100000000,
    "merging_chain_block_hash": "hash_here",
    "header": { /* block header fields */ }
  },
  "error": null,
  "id": 1
}
```

#### GetVirtualChainFromBlock

Returns the virtual chain from a given block hash:

```bash
$ curl -X POST http://127.0.0.1:16110 \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"GetVirtualChainFromBlock","params":["block_hash"],"id":1}'
```

Response:
```json
{
  "result": {
    "removed_chain_block_hashes": ["hash1", "hash2"],
    "added_chain_block_hashes": ["hash3", "hash4"],
    "accepted_transaction_ids": [
      {
        "accepting_block_hash": "hash3",
        "accepted_transaction_ids": ["tx1", "tx2"]
      }
    ]
  },
  "error": null,
  "id": 1
}
```

## Using the Kaspa WASM SDK  
  
The officially supported way to programmatically access kaspad is through the **Kaspa WASM SDK** for JavaScript and TypeScript environments. The SDK uses **wRPC** (WebSocket-based RPC) with Borsh or JSON encoding, which provides high-performance, real-time communication with the node.  
  
### Enabling wRPC on kaspad  
  
By default, wRPC is disabled in kaspad. Enable it with one of the following flags:  
  
```bash  
# JSON protocol (compatible with any WebSocket library)
kaspad --rpclisten-json=default  
  
# Borsh protocol (binary, higher performance)
kaspad --rpclisten-borsh=default  
```  
  
The `default` value uses the standard port for your network (mainnet: 18110 for JSON, 17110 for Borsh; testnet: 18210 for JSON, 17210 for Borsh).
  
### Node.js Usage  
  
For Node.js applications, install the Kaspa WASM SDK and use the `RpcClient` class:  
  
```javascript  
const { RpcClient, Encoding, initConsolePanicHook } = require('kaspa');  
  
// Optional: enable console panic hooks for debugging  
// initConsolePanicHook();  
  
// Create an RPC client  
const rpc = new RpcClient({  
    url: "127.0.0.1",  
    encoding: Encoding.Borsh,  
    networkId: "mainnet"  
});  
  
(async () => {  
    try {  
        await rpc.connect();  
          
        // Get server information  
        const info = await rpc.getInfo();  
        console.log("Server version:", info.serverVersion);  
        console.log("Network ID:", info.networkId);  
        console.log("Is synced:", info.isSynced);  
          
        // Get block count  
        const blockCount = await rpc.getBlockCount();  
        console.log("Block count:", blockCount);  
          
        // Get blockDAG info  
        const dagInfo = await rpc.getBlockDagInfo();  
        console.log("DAG info:", dagInfo);  
          
    } finally {  
        await rpc.disconnect();  
    }  
})();  
```  
  
The RpcClient supports both direct connections and automatic node resolution via the `Resolver` class.
  
### Browser Usage  
  
For web applications, import the SDK as an ES module:  
  
```html  
<html>  
<head>  
    <script type="module">  
        import * as kaspa from './kaspa/kaspa-wasm.js';  
          
        (async () => {  
            // Initialize the WASM module  
            await kaspa.default('./kaspa/kaspa-wasm_bg.wasm');  
              
            // Create an RPC client  
            const rpc = new kaspa.RpcClient({  
                url: "wss://public-node-url",  
                encoding: kaspa.Encoding.Borsh,  
                networkId: "mainnet"  
            });  
              
            try {  
                await rpc.connect();  
                  
                // Subscribe to notifications  
                rpc.addEventListener("connect", (event) => {  
                    console.log("Connected to", rpc.url);  
                });  
                  
                rpc.addEventListener("VirtualDaaScoreChanged", (event) => {  
                    console.log("DAA score changed:", event.data);  
                });  
                  
                await rpc.subscribeDaaScore();  
                  
                // Get server info  
                const info = await rpc.getInfo();  
                console.log(info);  
                  
            } finally {  
                await rpc.disconnect();  
            }  
        })();  
    </script>  
</head>  
<body></body>  
</html>  
```  
  
### Common RPC Methods  
  
The WASM SDK provides async methods for all RPC operations. Common methods include:  
  
- `getInfo()` - General node information
- `getSyncStatus()` - Synchronization status
- `getBlockCount()` - Number of blocks in the DAG
- `getBlockDagInfo()` - DAG structure information
- `getSink()` - Current tip block (highest cumulative difficulty)
- `getSinkBlueScore()` - Total work on the main chain
  
### Event Notifications  
  
The SDK supports server-side notifications through event listeners:  
  
```javascript  
// Subscribe to block additions  
await rpc.subscribeBlockAdded();  
rpc.addEventListener("BlockAdded", (event) => {  
    console.log("New block:", event.data);  
});  
  
// Subscribe to DAA score changes  
await rpc.subscribeDaaScore();  
rpc.addEventListener("VirtualDaaScoreChanged", (event) => {  
    console.log("New DAA score:", event.data.daaScore);  
});  
```  
  
### SDK Packages  
  
The WASM SDK is available in multiple variants for different use cases:  
  
- **kaspa** - Full SDK with WebSocket support (recommended for Node.js)  
- **kaspa-wasm** - Pure WASM module (for environments with native WebSocket)  

For more complex exploration, refer to the Kaspa RPC documentation for exact field names and structures.

---

### Official Resources

**rusty-kaspa** - The reference implementation of Kaspa, written in Rust.
- Repository: https://github.com/kaspanet/rusty-kaspa
- Documentation: https://docs.kaspa.org

---

## Summary

In this chapter, you learned how to:

1. **Download and compile kaspad** from source code using Rust
2. **Configure your Kaspa node** with pruning (default) or archival settings
3. **Run a full verification node** that independently validates all transactions
4. **Use the JSON-RPC API** on port 16110 to query blockDAG data programmatically
5. **Explore transactions and blocks** using both command-line tools and code
6. **Understand Kaspa-specific RPC methods** for blockDAG operations

If you followed the instructions in this chapter, you now have kaspad running and have begun exploring the Kaspa network and blockDAG using your own full node. From now on you can independently use software you control—on a computer you control—to verify that any kaspa you receive follow every rule in the Kaspa system without having to trust any outside authority.

In the coming chapters, we'll learn more about the rules of the system and how your node and your wallet use them to secure your money, protect your privacy, and make spending and receiving convenient.
