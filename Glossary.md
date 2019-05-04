---
layout: page
title: Glossary
permalink: /glossary/
nav_order: 7
---

# Glossary

### Admin Block

The admin block is subsumed by the Directory Block and contains signatures and organizational data about the Directory Block.

### Anchor

"The Merkle Root of a Directory Block in Factom that has been recordded into the Bitcoin Blockchain. All the data in Factom can be validated by finding teh collection of Anchors in the bitcoin blockchain, validating them, then confirming that all the Directory Blocks in Factom are, in fact, anchored to the Bitcoin Blockchain."<sup>1</sup>

### Audit Server

"A server running the exact same software as a Federate Server, but without the support required to be part of the Federated Server Pool. If a Federated Server in the Federated Server Pool goes off line, or steps out of the pool for maintenance, the Audit Server with the greatest support joins the Federated Server Pool."<sup>1</sup>

### Audit Server Pool

"The set of servers running the Federated Server software which publish a heartbeat, and have enough support to join the Audit Server Pool. The Audit Server Pool is of a fixed size." <sup>1</sup>

### Body Merkle Root

Body Merkle Root, or BodyMR, is produced when a Merkle tree is built from the body elements of a block and hashed. It is an ingredient in utilizing the KeyMR.

### Brainswap

The process of seamlessly moving an identity from one Factom node to another.

### Chain ID

"The name of a Factom Chain is hashed to create the ChainID. ChainID’s are used in the Directory Block, and anywhere else in the protocol that must refer to a Factom Chain." <sup>1</sup> The chain ID provides 

### Coinbase

The source of payments to compensate authority nodes in Factoids.

### Directory Block Signature

Directory Block signature, or DBSig, is sent at the conclusion of a Directory Block's 10 minute period by the Federated Server. It contains the server's identity chain ID, a public key, and a signature of the previous Directory Block's header.

### Directory Block

"This is the list of (ChainID, Entry Block Merkle Root) pairs collected by Factom over a 10 minute period. The Directory Block is sorted by ChainID. Directory Blocks are organized in a chain over time, and include the serial hash of the previous Directory Block."<sup>1</sup>

### Election

An election occurs when a Federated Server has faulted out and the network promotes an audit to take its place.

### End of Minute

End of Minute, or EOM, notifies other servers that there will be no more messages from that server and the next server can take over. <sup>2</sup>

### Entry

"A single data submission made by a user. A collection of entries makes up a Factom Chain. There are certain bookkeeping entries required, but there are no restrictions on the content of an Entry. An Entry is restricted to no more than 10K in size, and requires an Entry Credit for every 1K of data. Entry size is so restricted to insure reasonably fast propagation of Entries through the network, as required by the consensus algorithm. The user can string together multiple entries for larger content."<sup>1</sup>

### Entry Block

"The Entry Hashes of all entries for a particular Factom Chain received over a ten minute period. The Merkle Root for an Entry Block is stored in the Directory Block for that ten minute period."<sup>1</sup> Every Blocks may contain only entries belonging to a single chain. A Directory Block thus may refer to many Entry Blocks, organized by Chain ID.

### Entry Commit

"The submission of the Entry Hash along with the number of Entry Credits required to pay for the recording of the Entry. The Entry Commit must be signed by a valid Entry Credit Address with a balance of Entry Credits sufficient to cover the Entry Commit."<sup>1</sup>

### Entry Credit

"Factoids can be converted to Entry Credits, which act like a software license. They are tied to an Entry Credit Address. They are non­transferable, non­refundable, and can only be converted into Entries for the purpose of writing an entry into Factom."<sup>1</sup>

### Entry Credit Address

"A cryptocurrency address for holding Entry Credits, and authorizing Entry commits"<sup>1</sup>

### The Entry Credit Chain

"A Factom Chain maintained by the protocol that tracks all the Entry Credit balances of all the Entry Credits issued to public keys. This is one of the few protected chains that can only contain entries created by the Factom protocol."<sup>1</sup>

### Entry Hash

"The hash of an Entry, stored in the Entry Block for the ten minute period when the Entry was revealed."<sup>1</sup>

### Entry Reveal

"Submission for an Entry that matches a previous Entry Commit. No signature is required, as the hash of the Entry Reveal must match an existing Entry Commit." <sup>1</sup>

### Factoid

"The token issued by the protocol to the Factom Servers for collecting entries, creating and publishing the updated entry chains, and submitting the anchors to the Bitcoin blockchain."<sup>1</sup>

### Factoid Address

"A cryptocurrency address for securing Factoids. Only multi­signature transactions are supported, with a single signature being a special case (1 of 1)."<sup>1</sup>

### Factom Transaction

"Like a Bitcoin transaction, a Factoid transaction moves a balance of Factoids from the control of one Factoid Address to another Factoid Address"<sup>1</sup>

### Factom Chain

"Entries are written to a Factom Chain. A Factom Chain is composed of the first Entry which provides its Name, which is hashed to give its ChainID. Subsequent Entries must include this ChainID."<sup>1</sup>

### Federated Server

"A server running the Factom Server software which collects entries and Factoid transactions, validates them and commits them to their respective Factom Chains."<sup>1</sup>

### Federated Server Pool

"The collection of servers with the highest support that are currently in charge of the protocol. If any Federated Servers leave the pool, they are replaced with the highest ranking Audit Server."<sup>1</sup>

### Heartbeat

"Federated Servers are responsible for broadcasting a valid message within the Timeout Period (TOP). A special Heartbeat message can be broadcast should there be no valid communications to broadcast. Audit Servers must have a heartbeat broadcast every 10 TOPs."<sup>1</sup>

### Height

Height or DBHeight refers to the number of blocks since the Genesis Block (0).

###  Identity Chain ID

A collection of private keys created by a user, of which the public keys are combined into a chain name and a Chain ID is generated. Entry Credit keys are linked to that identity. The identity chain ID is that chain ID.

### Ignore

When a node is restarted it will ignore messages with timestamps prior to the node's start time. This gives the node time and resources to focus on complete booting. End of ignore does not necessarily mean the node is fully synced and up to date, however.

### Key Merkle Root

Key Merkle Root, or KeyMR, is a data structure that lets the user quickly validate a header. By producing a BodyMR and combining with the header, the user can validate the header with two hash operations. 

### Leader

See [Federated Server](#federated-server).

### Matryoshka Hash

A hashing of hashing of a secret random value maintained by a server and is maintained as part of its identity. Example: `hash(hash(hash(RandomValue)))`. The server is able to reveal layers of this information. Each reveal can be validated if the successive hash is known.

"This system allows the servers to inject deterministic, but unpredictable entropy into the selection process. It will prevent a single server from adjusting the state in order to influence the next state."<sup>1</sup>

### Merkle Tree

A Merkle tree is a data structure that combines hashes at each layer of the tree, with a root hash at the top of the tree called the Merkle root. See [Wikipedia](https://en.wikipedia.org/wiki/Merkle_tree) for details.

### Pause

A network pause occurs when a new Directoy Block has failed to be authored for an extended period of time. Related: In the factomd, the term stall means the node is trailing significantly behind the leaders. "More precisely, if a Minute lasts longer than 150% of the intended time (90 seconds on mainnet)."<sup>3</sup>

### Process List

"Each Federated Server broadcasts confirmations for entries to be added to Factom Chains. These are hashed and ordered, so that all listeners can validate the order and content of the process chain for a server. At the end of a minute, all Process Lists are executed to add entries to the Entry Blocks."<sup>1</sup> Effectively, the Process List is the current block being built.

### Protected Chain

"A Factom Chain which does not allow invalid entries to be entered. At this time, the Factoid Chain and the Entry Credit Chain are the only two protected chains in Factom. They are protected against invalid entries because the protocol requires the entries to be valid in order to properly function."<sup>1</sup>

### Random Deterministic Seed

"A Random number generated at the end of each minute with a provably correct value used to order processes in Factom."<sup>1</sup>

### Server Add Message

A Federated Server will issue a Server Add Message when an audit server is to be added to the federated server list.

### Server Reject Message

When a Federated Server has observed that a majority of Federated Servers have issued a Server Fault Message on a given server, the Federated Server issues a Server Reject Message on that server.<sup>1</sup> This removes the server from the Federated Server Pool.

### Server Fault Message

"A message broadcast by a Federated Server to indicate a fault with a Federated or Audit server."<sup>1</sup> This message will time out or the server may issue a retraction with a Server Fault Repair Message.

### Server Fault Repair Message

"A message broadcast by a Federated Server to retract a previous Server Fault Message."<sup>1</sup>

### Timeout Period

The timeout out period, or TOP, required for every server to broadcast, or a Server Fault Message will be issued, and teh Federated SErver will be dropped from the Federated Server Pool. Audit Servers must broadcast once every 10 TOP. See <sup>1</sup>

### Virtual Machine

While executing the [Process List](#process-list), subsets of entries to be added to Entry Blocks are assigned to VMs (which you can think of as simply a list). Each Federated Server takes responsibility for a given VM. More information about the role of VMs will be included in the section on consensus.


## References

1. [Factom Ledger by Consensus](https://github.com/FactomProject/FactomDocs/blob/master/FactomLedgerbyConsensus.pdf) by Paul Snow et al., January 17, 2015.

2. [docs.factom.com](https://docs.factom.com/).

3. [Glossary Forum Post](https://factomize.com/forums/threads/glossary.1668/), by WhoSoup, February 19, 2019.