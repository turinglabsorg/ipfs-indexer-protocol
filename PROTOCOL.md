# IPFS Indexer Protocol

This protocol is still a WIP, comments are very appreciated.

## Problem

IPFS lacks the ability to search inside the huge amount of existing data, you must know exactly where a file is to get it back. This means there's no way to search inside it. Since the decentralised web is getting more attention, we expect the next years more and more people will start hosting applications and protocols using IPFS. But like the pre-google internet there's no simple way to discover contents.

## Goal

The protocol aims to create a collaborative, open-source way to track (and search) relevant data on IPFS. Let's say the main goal is to create the "Google of IPFS", of course taking only the good parts. Service will not track users and will remain privacy-oriented due to the simple fact that the entire database is shared between protocol's nodes.

Of course this protocol will not guarantee data retrievability or such, which can be achieved using other on-chain protocols like [Retrieval Pinning](https://www.notion.so/pl-strflt/Retrieval-Pinning-Service-252f52f47b3944e8a54e8d0f553c5cd7).

Unlike [Filecoin Network Indexer](https://docs.cid.contact/filecoin-network-indexer/overview) our goal is to search "all CIDs" containing a specific word. When we have found some interesting CID we can ask the Filecoin Network Indexer where actually retrieve the file.

## Protocol overview

The protocol is made up of many different parts:

- **[IPDB](https://github.com/turinglabsorg/ipdb)** which is the database where the protocol will store informations, it's an on-chain database and will be integrated inside the **smart contract**. An extension of IPDB for full-text search will be developed to help the frontend easily search in the database. Database is a simple key/value store so we'll link `original_cid` to `metadata_cid` and only those two informations will go on-chain. Actual `metadata` will be retrieved directly from IPFS using `metadata_cid`.

- A **smart contract** (which will be live in an EVM) that coordinates the work of the protocol's nodes and allows users to ask for active indexing (by paying a fee to the protocol).

- **Clients** who will ask for active indexing of files by providing an *onchain* transaction and a *payment*. The amount needed to ask for an indexing can be proportioned to the **size** of the file (or files) to scan which directly results in the time needed to scan a file.
 
- A **network of nodes**, which are the core of the protocol, that will listen to the smart contract for active indexing. When some user asks for indexing the protocol will retrieve the data and will download the metadata from the file (or folder). Then nodes will try to find a consensus on the metadata extracted (by creating a new json file). After an `n` of `m` nodes found and extracted same metadata an *onchain* transaction of aggregate signatures will be sent to the contract and the values will be stored.

- A **dApp** that will be the public frontpage of the protocol, specifically designed to work independently by anyone who wants to download entire database and make a search directly on its own computer. Each search will be done *for free* because informations are actually stored and replicated on your local machine. We expect of course to expose a public website.