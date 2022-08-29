# IPFS Indexer Protocol

This protocol is still a WIP, comments are very appreciated.

## Problem

IPFS lacks the ability to search inside the huge amount of existing data, you must know exactly where a file is to get it back. This means there's no way to search inside it. Since the decentralised web is getting more attention we expect the next years more and more people will start host applications and protocols using IPFS. But like the pre-google internet there's no simple way to discover contents.

## Goal

The protocol aims to create a collaborative, open-source way to track (and search) relevant data on IPFS. Let's say the main goal is to create the "Google of IPFS", of course taking only the good parts. Service will not track users and will remain privacy-oriented due to the simple fact that entire database is shared between protocol's nodes.

Of course this protocol will not guarantee data retrievability or such, which can be achieved using other on-chain protocols like [Retrireval Pinning](https://www.notion.so/pl-strflt/Retrieval-Pinning-Service-252f52f47b3944e8a54e8d0f553c5cd7).

## Protocol overview

Protocol is made by many different parts:

- **[IPDB](https://github.com/turinglabsorg/ipdb)** which is the database where the protocol will store the informations, it's an on-chain database and will be integrated inside the **smart contract**. An extension of IPDB for full-text search will be developed to help the frontend easily search in the database.

- A **smart contract** (which will be live in an EVM) that coordinates the work of the protocol's nodes and allow users ask for active indexing (by paying a fee to the prototcol).

- **Clients** which will ask for active indexing of files by providing an *onchain* transaction and a *payment*. The amount needed to ask for an indexing can be proportioned to the **size** of the file (or files) to scan which directly traduces in the time needed to scan a file.
 
- A **network of nodes** which are the core of the protocol, they will listen to the smart contract for active indexing. When some user asks for indexing the protocol will retrieve the data and will download the metadata from the file (or folder). Then nodes will try to find a consensus on the metadata extracted (by creating a new json file). After an `n` of `m` nodes found and extracted same metadata an *onchain* transaction of aggregate signatures will be sent to the contract and the values will be stored.

- A **dApp** that's the public frontpage of the protocol (where anyone can search basically) and should be designed to work independently by anyone wants to download entire database and make searchs directly on it's own computer.