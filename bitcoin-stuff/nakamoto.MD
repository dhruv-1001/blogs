# CBF client implementation - Nakamoto

Compact block filters are a condensed representation of the contents of a block that allow wallets to determine whether the block contains any transactions involving the user's keys. A full node uses [BIP158](https://github.com/bitcoin/bips/blob/master/bip-0158.mediawiki) to create a Golomb-Rice Coded Set (GCS) of the data from each block in the block chain. 

[Nakamoto](https://github.com/cloudhead/nakamoto) is a library written in rust that implements CBF. This blog tries to cover [Nakamoto](https://github.com/cloudhead/nakamoto)'s implementation of client side CBF described in [BIP 157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki)


## Blog overview
- **Why do we need Compact Block Filters** - This section explains why do we use Compact Block Filters
- **How to use Nakamoto** - This section covers how to use the Nakamoto library
- **CBF implementation - Nakamoto** - This section covers how Nakamoto implements CBF and it's architecture

## Why do we use Compact Block Filters
Let's first discuss what we had before the Compact Filters. Well there was Bloom Filtering ([BIP0037](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki)). Motivation for these filters were that, as Bitcoin grows in usage the amount of bandwidth needed to download blocks and transaction broadcasts increased and the clients implementing SPV do not attempt to fully verify the block chain, instead just checking that block headers connect together correctly and trusting that the transactions in a chain of high difficulty are in fact valid.

The motivation for the Compact Block Filters Boils down to SPV providing zero privacy to wallets and other applications, also malicious full nodes serving SPV light clients can omit critical data with little risk of detection, which is unacceptable for some applications that must respond to certain on-chain events. Also honest nodes servicing BIP-37 light clients may incur significant I/O and CPU resource usage due to maliciously crafted Bloom filters.

To solve this issue, instead of the client sending a filter to a full node peer, full nodes generate deterministic filters on block data and the same set of data is served to all clients. A light client can then download an entire block if the filter matches the data it is watching for. This keeps the computation required to serve filters minimal, and eliminates the I/O asymmetry that makes BIP-37 enabled nodes vulnerable. And these are the compact block filters ([BIP 157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki)) that are generaged at nodes and served to clients.

## How to use Nakamoto library
Here I will be covering on how to clone, build and use nakamoto on our own. First you'll obviously need git, rust, cargo all that good stuff intalled as we are going to clone & build a rust library. 

Open up the terminal and set your directory to where you would like to clone the repository. And then clone the repisitory from github
```
$ git clone https://github.com/cloudhead/nakamoto
```
Change directory into the Nakamoto
```
$ cd nakamoto
```
Next part would be to build the nakamoto-wallet and get the library running.
```
$ cargo build --release -p nakamoto-wallet
```
This should build the `nakamoto-wallet` executable file. To check this executable enter this command and you should be able to locate the generated executable file.
```
$ ls target/release 
build                   examples                libnakamoto_wallet.d    nakamoto-wallet
deps                    incremental             libnakamoto_wallet.rlib nakamoto-wallet.d
```
Now, run the executable `nakamoto-wallet`
```
$ cargo run --release -p nakamoto-wallet
Required options not provided:
    --genesis
```
It requires us to mention the block we want to start our scan from. In this case let's do the first block ever.
```
$ cargo run --release -p nakamoto-wallet --genesis 0
2022-08-18T04:37:06.952+05:30 [nakamoto_wallet] Fatal: at least one address must be specified with `--addresses`
```
And, ofcourse we also require addresses that we would like to track. In this case i'm using a single address. So our command would now look somewhat like this.
```
$ cargo run --release -p nakamoto-wallet --genesis 0 --addresses "bc1qa47tl4vmz8j82wdsdkmxex30r23c9ljs84fxah" 
```
This would generate soo much text, something like this
```
2022-08-06T14:02:57.616+05:30 [nakamoto_wallet] Waiting for peers..
2022-08-06T14:02:57.617+05:30 [nakamoto_client::client] Initializing client (Mainnet)..
2022-08-06T14:02:57.617+05:30 [nakamoto_client::client] Genesis block hash is 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
2022-08-06T14:02:57.617+05:30 [nakamoto_client::client] Initializing new block store "/Users/dhruvbaliyan/.nakamoto/mainnet/headers.db"
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Initializing block filters..
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Initializing new filter header store "/Users/dhruvbaliyan/.nakamoto/mainnet/filters.db"
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Verifying filter headers..
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Loading peer addresses..
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Initializing new peer address cache "/Users/dhruvbaliyan/.nakamoto/mainnet/peers.json"
2022-08-06T14:02:57.619+05:30 [nakamoto_client::client] Address book is empty. Trying DNS seeds..
2022-08-06T14:03:06.421+05:30 [nakamoto_client::client] 212 seeds added to address book
2022-08-06T14:03:06.425+05:30 [nakamoto_net_poll::reactor] Initializing protocol..
... and much much more
```

This progess could take an hour or two to catchup with the blockchain as it is now. Now what is actutally happening here? What is all this text? In next topic i'll try to explain how the nakamoto works.

## CBF implementation - Nakamoto
Let's first checkout what does [BIP0157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki) recommends on how to handle the [client operations](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki#client-operation) in CBF. This is how,

### Client Operation BIP0157 Recommendation
Clients SHOULD first sync the entire block header chain from peers using the standard headers-first syncing mechanism before downloading any block filters or filter headers. Clients configured with trusted checkpoints MAY only sync headers started from the last checkpoint. Clients SHOULD disconnect any outbound peers whose best chain has significantly less work than the known longest proof-of-work chain.

Once a client's block headers are in sync, it SHOULD download and verify filter headers for all blocks and filter types that it might later download. The client SHOULD send getcfheaders messages to peers and derive and store the filter headers for each block. The client MAY first fetch headers at evenly spaced intervals of 1,000 by sending getcfcheckpt. The header checkpoints allow the client to download filter headers for different intervals from multiple peers in parallel, verifying each range of 1,000 headers against the checkpoints.

Unless securely connected to a trusted peer that is serving filter headers, the client SHOULD connect to multiple outbound peers that support each filter type to mitigate the risk of downloading incorrect headers. If the client receives conflicting filter headers from different peers for any block and filter type, it SHOULD interrogate them to determine which is faulty. To determine which peer is wrong the client SHOULD use getcfheaders and/or getcfcheckpt to first identify the first filter headers that the peers disagree on. The client then SHOULD download the full block from any peer and derive the correct filter and filter header. The client SHOULD ban any peers that sent a filter header that does not match the computed one.

Once the client has downloaded and verified all filter headers needed, and no outbound peers have sent conflicting headers, the client can download the actual block filters it needs. The client MAY backfill filter headers before the first verified one at this point if it only downloaded them starting at a later point. Clients SHOULD persist the verified filter headers for last 100 blocks in the chain (or whatever finality depth is desired), to compare against headers received from new peers after restart. They MAY store more filter headers to avoid redownloading them if a rescan is later necessary.

Starting from the first block in the desired range, the client now MAY download the filters. The client SHOULD test that each filter links to its corresponding filter header and ban peers that send incorrect filters. The client MAY download multiple filters at once to increase throughput, though it SHOULD test the filters sequentially. The client MAY check if a filter is empty before requesting it by checking if the filter header commits to the hash of the empty filter, saving a round trip if that is the case.

Each time a new valid block header is received, the client SHOULD request the corresponding filter headers from all eligible peers. If two peers send conflicting filter headers, the client should interrogate them as described above and ban any peers that send an invalid header.

If a client is fetching full blocks from the P2P network, they SHOULD be downloaded from outbound peers at random to mitigate privacy loss due to transaction intersection analysis. Note that blocks may be downloaded from peers that do not support this BIP 157/158.

### Client Operation Nakamoto
Nakamoto almost follows the [BIP 157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki) recommendations, except that the nakamoto thinks that it is impractical to sync the full block header chain before syncing any filter headers. So, they chose a middle-ground, nakamoto starts syncing the filter headers once the blocker headers have synced upto a certain checkpoint and not till the very recent block.

### Namamoto working
Let's start from the example given by nakamoto that shows us on how to use the library. That piece of code is inside `nakamoto/wallet/src/lib.rs`.
```
// these are the arguments that we provided through the CLI in the example above.
pub fn run(addresses: Vec<Address>, birth: Height) -> Result<(), Error> {
    let cfg = Config {
        listen: vec![], // Don't listen for incoming connections.
        protocol: protocol::Config {
            network: Network::Mainnet,
            ..protocol::Config::default()
        },
        ..Config::default()
    };

    // Create a new client using `Reactor` for networking.
    let client = Client::<Reactor>::new()?;
    let handle = client.handle();

    // Create a new wallet and rescan the chain from the provided `birth` height for
    // matching addresses.
    let mut wallet = Wallet::new(handle.clone(), addresses);

    // Start the network client in the background.
    thread::spawn(|| client.run(cfg).unwrap());

    wallet.rescan(birth)?;

    log::info!("Balance is {} sats", wallet.balance());
    log::info!("Rescan complete.");

    handle.shutdown()?;

    Ok(())
}
```
This method creates a `Client` and a client handle that is passed to the `Wallet` which listens to the events caused by the client, and logs them to the screen. Logs here contains information like balance & syncing update.

#### Client
So the main work is going on under the `Client` which is working on a different thread. On constructing the running the `Client`, it first looks for previously stored filters, headers, and peers on your local machine, and if there aren't any the `Client` will then create files for filters, headers and peers, and to initially connect to some peers it will flush the DNS seeds into the peers which will return other peers on being asked throught a `GetAddr` request.

The client then creates `Protocol` passing the data from filters, headers and peers, and then run the `Reactor` passing the `Protocol` created.

#### Protocol
On being created the `Protocol` initializes different managers like the `SyncManager`, `PingManager`, `FilterManager`, `PeerManager`, `AddressManager` & `InventoryManager`.
The task of the `Protocol` is basically to orchestrate all the managers and let them work together. For example asking `PeerManager` to fetch more peers using `Peer` interface, or to order the `InventoryManager` to store the new fetched headers of filters found.

#### Reactor
This file is the backbone of the library. When the `Client` is created it also creates different subscribers and publishers, which basically listens and emits events respectively. The publishers are then passed to different `Managers` the are the source of events (events like `connecting to peer`, `peer disconnected`). And the subscribers are passed to the `Reactor` which listens to those events and then react accordingly by passing the command to the `Protocol`, the `Protocol` then acts according the event and tries to follow the BIP0157 recommendation.
Example for such an event loop would be, the `PeerManager` reports that a `peer has disconnected`, the `Reactor` will listen to this event and pass on this information to the `Protocol`, the `Protocol` then checks if the number of connected peers are below threshold, if yes the `Protocol` orders the `PeerManager` to connect to more peers, which will then create another event `connecting to peer` and `Reactor` will listen to this again creating an event loop.

### About the architecture 
![](https://raw.githubusercontent.com/cloudhead/nakamoto/66d0507896a4864dda3c4b0d08e30ef94c8d7a19/docs/architecture.svg)

As explained above, it starts from `Client` creating the `Reactor`, and the `Protocol` manages all the Managers and ensure that BIP0157's recommendtaions are followed. You can checkout the individual managers and see what functions do they have, and how they work.

So it's basically an event based architecture where `Reactor` listens for all the events and then pass on the processing accordingly. This is done by the single-threaded reactor that uses minimal non-blocking I/O library `Popol`.

## References
[BIP0037 - Connection Bloom filtering](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki)
[BIP0157 - Client Side Block Filtering](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki)
[BIP0158 - Compact Block Filters for Light Clients](https://github.com/bitcoin/bips/blob/master/bip-0158.mediawiki)
[Nakamoto - CBF Implementation in Rust](https://github.com/cloudhead/nakamoto)