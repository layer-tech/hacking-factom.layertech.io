---
layout: page
title: Peer to Peer
permalink: /peer-to-peer/
nav_order: 4
---

# Peer to Peer
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introduction

Factom spreads information to all nodes connected to its network by passing messages from neighbor to neighbor. In this chapter we will discuss the story of how P2P is initialized, how it gets those neighbors, and how the mechanics of how it sends messages to its neighbors.

![Factom P2P](https://raw.githubusercontent.com/FactomProject/factomd/master/p2p/diagram.jpg "Factom P2P")

(Chart from factomd/p2p. See <sup>1</sup>.)

## Controller

The P2P code is autonomously operating, separate from the rest of the factomd codebase. The Controller ([`p2p/controller.go`](https://github.com/FactomProject/factomd/blob/master/p2p/controller.go)) contains the main loop for the P2P package that keeps factomd connected to the network--managing ingress and egress connections, messages, and discovering peers.

The controller is initiated in `engine/NetStart.go` where a [`ControllerInit`](https://github.com/FactomProject/factomd/blob/master/p2p/controller.go#L66) is populated with things like the `SeedURL`, `ConfigPeers`, `Exclusive`, and `Network`. 

Sourcing initial peers is handled during this start up. These sources follow: `SeedURL` is a URL found in the `factomd.conf` file, classified by network type. If absent the `SeedURL` is found coded directly into the [State.LoadConfig](https://github.com/FactomProject/factomd/blob/4757ff64d32155d4fa02ea01e4e5d712c828eeb9/state/state.go#L715) function. A `PeersFile` is also set where peers are loaded and saved; by default it is set to `peers.json`. 

There is a third source of peers called Special Peers which are always loaded, maintained, and saved. The Controller derives Special Peers from the config (and command line) under the property `MainSpecialPeers` or `TestSpecialPeers`, or as appropriate according to network. Regardless of which peers are later saved or reloaded, Special Peers are never deleted by the Controller.

Controller uses this information to start and maintain connections to peers, and it calls on Discovery to do this.

## Discovery

The `SeedURL` and `PeersFile` are passed to [`p2p/discovery.go`](https://github.com/FactomProject/factomd/blob/master/p2p/discovery.go). Discovery contains a `knownPeers` map that stores Peers by their hash. Each Peer is defined as:

```go
type Peer struct {
	QualityScore int32     // 0 is neutral quality, negative is a bad peer.
	Address      string    // Must be in form of x.x.x.x
	Port         string    // Must be in form of xxxx
	NodeID       uint64    // a nonce to distinguish multiple nodes behind one IP address
	Hash         string    // This is more of a connection ID than hash right now.
	Location     uint32    // IP address as an int.
	Network      NetworkID // The network this peer reference lives on.
	Type         uint8
	Connections  int                  // Number of successful connections.
	LastContact  time.Time            // Keep track of how long ago we talked to the peer.
	Source       map[string]time.Time // source where we heard from the peer.

	// logging
	logger *log.Entry
}
```

On initiating, Discovery reaches out to the `SeedURL` to add peers to its `knownPeers` map assuming none of the peers are bad. At time of writing, loading peers from disk is commented out but this would also occur during initiation of Discovery.

Discovery also contains the means by which the node selects peers to share with other hosts. Discovery filters out this node's Special Peers, filters by `QualityScore` and shares with the network in order to only share quality peers.

## Run Loop

After Controller has initiated, `engine/NetStart.go` calls on the Controller's `StartNetwork` function which begins the `runloop` goroutine, starts the `acceptLoop` to listen for new connections, and dials Special Peers. `runloop` will loop until the `keepRunning` property is false.

>> Take a moment to check out [`p2p/protocol.go`](https://github.com/FactomProject/factomd/blob/master/p2p/protocol.go) to reference some important constant values and relatons.
{: .fw-300 }

The loop processes commands, routes messages, manages peers, and updates metrics.

### Executes Commands

Because runloop is isolated, it relies on commands sent over the `commandChannel` to know to perform certain actions. An overview of commands that runloop processes include:

* Dial peer (creates a new connection)
* Add peer (sent from Accept loop)
* Shutdown (sets `keepRunning` false)
* Adjust Peer quality
* Ban peer
* Disconnect peer

For example, the `acceptLoop` in Controller (which listens for new connections and pings them back to test them) passes the AddPeer command via the protocol.go's `BlockFreeChannelSend` function to let `runloop` know to add a new peer.

### Manages Peers

Over the course of the operation, peers might disconnect or shut down and new peers might arrive or some peers may start to degrade in quality. This makes a list of things to get done.

* If it's been a while since we last discovered peers, run Disovery's `DiscoverPeersFromSeed`
* Attempt to fill up egress connection slots by dialing a subset of peers
* Have Discovery save peers list to disk
* Create a peer request parcel and send to all connections

### Routes Messages

`runloop` also routes messages (we will call them parcels at this level) by calling the `route()` function inside the Controller. The application builds a collection of parcels that need to be sent to specific peers, or broadcast to all peers. `route()` begins by passing parcels from peer to the application, then gathers up the outgoing parcels.

Parcels contain flags to include specific peers, broadcast to all peers (or many peers, depending on bool param), or send to random peer. The parcel is then placed on the `sendChannel` for outgoing parcels handled in `p2p/connection.go`.

## Connection

[`p2p/connection.go`](https://github.com/FactomProject/factomd/blob/master/p2p/connection.go) directly handles parcels that will received and sent out to the network. It contains the [`Connection`](https://github.com/FactomProject/factomd/blob/master/p2p/connection.go#L28) struct (note `conn`, `encoder`, and `decoder`).

When a connection to a peer is made, `connection.Start()` is called on that connection. This starts [`runLoop()`](https://github.com/FactomProject/factomd/blob/master/p2p/connection.go#L230) for this connection. This loop executes until the connection is shut down; it is the only control where `Connection` struct can be updated, including the `state` property which closes the goroutines it fires off. The goroutines it starts includes the `processSends()` and `processReceives()` loops.

Let's look at [`processSends()`](https://github.com/FactomProject/factomd/blob/master/p2p/connection.go#L393). It checks the `sendChannel` channel for outgoing parcels on every loop iteration. Two possibilities here include handling a connection parcel or a connection command. For handling a parcel, it sends the parcel along to `sendParcel()`.

`sendParcel()` places a write deadline on the connection (see the `p2p/protocol.go` for constant it relies on) and relies on [`gob.Encode()`](https://golang.org/pkg/encoding/gob/#Encoder.Encode) to send the parcel to the connection.

(`p2p/connection.go` handles more than the functions described here, but we will leave those as an exercise for the reader.)

## Parcels

When we started talking about how P2P routes messages, we introduced the term "Parcel".

```go
// Parcel is the atomic level of communication for the p2p network.  It contains within it the necessary info for
// the networking protocol, plus the message that the Application is sending.
type Parcel struct {
	Header  ParcelHeader
	Payload []byte
}
```

A parcel header contains a lot of important information and it is worth stopping now to read over the structs in [`p2p/parcel.go`](https://github.com/FactomProject/factomd/blob/master/p2p/parcel.go) to get an idea of what sort of behaviors a parcel will invoke.

When the application wants to send a message to the network, it converts the payload into a byte slice and passes it to `ParcelsForPayload(NetworkId, []byte) []Parcel` which returns a collection of parcels. `ParcelsForPayload` divides the payload into lengths of `MaxPayloadSize` (note: this is a very large value) and creates `Parcel`s, indexing each parcel part and placing that index into the `ParcelHeader.PartNo` property. The application sends messages via parcels.

When Connection processes incoming parcels, `parcel.go` and `parts_assembler.go` work together to track and reassmble the partial messages based on that header information. If a parcel is received but remaining parts are not received before [`MaxTimeWaitingForReassembly`](https://github.com/FactomProject/factomd/blob/master/p2p/parts_assembler.go) times out, the messages are deleted.

## P2PProxy: How the Application talks to P2P Layer

As mentioned, P2P is isolated from the rest of factomd. There must be an intermediary to move from the messages that factomd moves around and the lower level details handled in P2P. Factom relies on [`engine/p2pProxy.go`](https://github.com/FactomProject/factomd/blob/master/engine/p2pProxy.go) to be the middleman.

Continuing our story of following how message are sent out to the network, consider `Send(msg interfaces.IMsg) error` in `engine/p2pProxy.go`.  If the message hash is not nil, among other things, a message is formed from the [`FactomMessage`](https://github.com/FactomProject/factomd/blob/master/engine/p2pProxy.go#L48) struct. The message itself is included in the Message field. Depending on the intended recipient (full broadcast, partial broadcast, or random peer) the message is flagged appropriately, placed on the `f.BroadcastOut` channel.

A goroutine is started by `StartProxy()` calls `ManageOutChannel()`. This function pulls messages found in `f.BroadcastOut` channel, wraps the message into parcels, and places on the `f.ToNetwork` channel. This channel is what the P2P Controller pulls messages from to begin routing.

[Next Chapter](/hackingfactom/operations){: .btn .btn-blue }

## Additional Reading

[Factom Protocol P2P: A TCP Gossip Network](https://factomize.com/factom-protocol-p2p-a-tcp-gossip-network/) by WhoSoup on March 28, 2019.

## References

1. [README.md factomd/p2p](https://github.com/FactomProject/factomd/blob/master/p2p/README.md) on factomd github repo.