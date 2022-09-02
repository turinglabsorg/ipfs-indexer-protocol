# IPFS Indexer Protocol

This protocol is still a WIP, comments are very appreciated.

## Problem

IPFS lacks the ability to search inside the huge amount of existing data, you must know exactly where a file is (by knowing the *ContentID* or *CID*) to get it back. This means there's no way to search inside it. 

Since the decentralised web is getting more attention, we expect the next years more and more people will start hosting applications and protocols using IPFS. But like the pre-google internet there's no simple way to discover contents.

## Goal

The protocol aims to create a collaborative, open-source way to track (and search) relevant data on IPFS. Let's say the main goal is to create the "Google of IPFS", of course taking only the good parts. Service will not track users and will remain privacy-oriented due to the simple fact that the entire database is shared between protocol's nodes.

Of course this protocol will not guarantee data retrievability or such, which can be achieved using other on-chain protocols like [Retrieval Pinning](https://www.notion.so/pl-strflt/Retrieval-Pinning-Service-252f52f47b3944e8a54e8d0f553c5cd7).

Unlike [Filecoin Network Indexer](https://docs.cid.contact/filecoin-network-indexer/overview) our goal is to search "all CIDs" containing a specific word. When we have found some interesting CID we can ask the Filecoin Network Indexer where actually retrieve the file.

## Protocol overview

The protocol is made up of many different parties:

- **[IPDB](https://github.com/turinglabsorg/ipdb)** which is the database where the protocol will store informations, it's an on-chain database and will be integrated inside the **smart contract**. An extension of IPDB for full-text search will be developed to help the frontend easily search in the database. Database is a simple key/value store so we'll link `original_cid` to `metadata_cid` and only those two informations will go on-chain. Actual `metadata` will be retrieved directly from IPFS using `metadata_cid`.

- A **smart contract** (which will be live in an EVM) that coordinates the work of the protocol's nodes and allows users to ask for active indexing (by paying a gas fee to the protocol).

- **Clients** who will ask for active indexing of files by providing an *onchain* transaction and an *index_fee*. The amount needed to ask for an indexing is the result of *multiplication* between the **size** of the file (or files) and the *index_price*.
 
- A **network of nodes** aka **oracle**, which is the core of the protocol, that will listen to the smart contract for active indexing. When some user asks for indexing the   will retrieve the data and will download the metadata from the file (or folder). Then nodes will try to find a consensus on the metadata extracted (by creating a new json file). After an `n` of `m` nodes found and extracted same metadata an *onchain* transaction of aggregate signatures will be sent to the contract and the values will be stored. Threshold is a parameter inside the **smart contract** and if threshold is not met (for ex. because data is not retrievable) before the `indexing_request_expiration` the user will need to run a second transaction.

- A **dApp** that will be the public frontpage of the protocol, specifically designed to work independently by anyone who wants to download entire database and make a search directly on its own computer. Each search will be done *for free* because informations are actually stored and replicated on your local machine. We expect of course to expose a public website.


## Smart contract specifications

### Parameters

- `indexing_request_expiration`: Time that the oracle has to find a consensus about extracted metadata.
- `consensus_threshold`: Amount of signature required to successfully index a file inside the protocol.
- `index_price`: Amount in *wei* to store informations inside the protocol.
- `max_block_size`: Amount of documents that can be stored in one single *IPDB* block, this parameter is used by oracle to decide after how much time store a new block on IPDB.

### Methods

`CreateIndexRequest{value: index_fee}(string _cid)`: 

Method which will allow anyone create an index request. Value of the transaction is the result of the multiplication between *file_size* and `index_price`. Each request will get its own id and this id is emitted.

`ConfirmIndex(uint256 _id_request, string _metadata_cid, bytes[] _signatures)`: 

Method which will allow oracle store the `metadata_cid` by sending at least `consensus_thresold` signatures.

`StoreBlock(string _database_cid, uint256 _block, bytes[] _signatures)`:

Method which will allow oracle store latest version of the database inside *IPDB* in a specific sector called `block`. This block can, eventually, be overwritten if there are errors when a new block is produced.

### Events

`IndexRequestCreated(account _sender, string _cid, uint256 _id_request)`:

Event emitted when an account creates a new index request.

`NewBlockNeeded(uint256 _nextblock)`:

Event emitted when `max_block_size` is met and oracle needs to emit a new block on *IPDB*.

`NewBlockStored(uint256 _block)`:

Event emitted when a new block is produced and stored on *IPDB*.

## Index flow

1) An *user* emits sends a transaction by running `CreateIndexRequest`.
2) An `IndexRequestCreated` event is emitted to network.
3) Oracle will listen the event and will start search for the original file.
4) If the original file is found it is scanned and `metadata` are extracted.
5) After the *oracle* found a consensus will send a `ConfirmIndex` transaction.
6) (optional) If the `max_block_size` is met the *oracle* will need to send a `StoreBlock` transaction before continue confirm indexes. 