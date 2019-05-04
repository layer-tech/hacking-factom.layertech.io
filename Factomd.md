---
layout: page
title: Factomd
permalink: /factomd/
nav_order: 2
---


# Factom Daemon
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Start Up

[`factomd.go`](https://github.com/FactomProject/factomd/blob/master/factomd.go) is the entry point for the program. It starts by defining `state := Factom(params, sim_Stdin)`. And as long as `state.Running()` is true, the program remains running. `Factomd` is found in [`engine/factomd.go`](https://github.com/FactomProject/factomd/blob/master/engine/factomd.go).

The first order of business is to initialize a new [`interfaces.IState`](https://github.com/FactomProject/factomd/blob/master/common/interfaces/state.go#L30) object and set its `IsRunning` property to true. From there, additional items are set such as the timestamp at boot, a message filter timestamp, and a new elections factory--and then it starts the networking system.

### NetStart

Networking is started at `engine/NetStart.go`. NetStart handles a lot of things, though, including but not limited to:

* enables or disables control panel
* registers Prometheus handlers
* sets ports to open 
* checks if RPC credentials are required
* runs `CheckChainHeads` 
* runs `FixChainHeads`
* starts pprof with `StartProfiler` (see  ["Operations"](/operations) chapter)
* sets log levels

> Many of the things that control the flow of start up and set important pieces of state are found in [`engine/factomParams.go`](https://github.com/FactomProject/factomd/blob/master/engine/factomParams.go). Stop and take a brief look at the `init()` function and the defaults. 
{: .fw-300 }

> It is equally important to check out the [`State.Init()`](https://github.com/FactomProject/factomd/blob/master/state/state.go#L887) function which is run during this initialization prior to starting the server.
{: .fw-300 }

Then, a few hundred lines later,

```go
//************************************************
// Actually setup the Network
//************************************************
```


If we are running a single full mainnet node, we will call `makeServer` once and supply it with a copy of the state object that we have been building up. It creates a new [FactomNode](https://github.com/FactomProject/factomd/blob/5f787fc9bf067dac6df2c36803e03e2a1a3b3186/engine/NetStart.go#L37) struct and returns it. If we are running a simulated network, additional Factom servers are started at this point and given skeleton Identities. Then, the P2P network is started.

The type of network we are on is determined (main, test, local, custom). The `SeedURL` is determined here based on the type of network we are on--the role of `SeedURL` will be discussed in the ["Peer to Peer Network"](/peer-to-peer) chapter. The most important module for peer-to-peer networking in Factom is started here, the `p2p.Controller`, also discussed in the ["Peer to Peer Network"](/peer-to-peer) chapter.

For a simulated network, the pattern of network is set here. This includes network options like "circle" and "tree"--the deafult is "alot+". These different configurations of how nodes are shaped and connected helps evaluate performance of the gossip protocol. The `arborjs.org/halfviz` visualization text of the network is generated here, you can copy it into the arborjs page to visualize the network pattern. Here is an example of what halfviz does:

![halfviz](/hackingfactom/assets/halfviz.png "halfviz")

(It is possible to have factomd keep a journal of all messages it receives, and replay those messages, if the paramters are set--by default this is disabled. But if it is enabled, factomd will make note of that fact at this point.)
{: .fw-300}

Prometheus (see ["Operations"](/operations) chapter) is started and made available on port `9876`. The control panel server is started. The web server API is started (see  ["Operations"](/operations) chapter). And finally the simulator listener is started for listening on stdin.

The servers that have been made are started in the `startServers` function call.

## A Factom Server is Born

Inside `NetStart.go` the function `startServers` fires 6 goroutines for our new Factom node.

* `NetworkProcessorNet`
* `Timer(fnode.State)`
* `state.LoadDatabase(fnode.State)`
* `fnode.State.GoSyncEntries()`
* `elections.Run(fnode.State)`
* `fnode.State.ValidatorLoop()`

State is aptly named for its role in keeping information about the node during its start up, operation, and shut down.

### NetworkProcessorNet

This call goes to [`engine/NetworkProcessorNet.go`](https://github.com/FactomProject/factomd/blob/master/engine/NetworkProcessorNet.go) and takes the `FactomNode` as an argument, which contains the State. The node is passed to goroutines that call `Peers` and `NetworkOutputs` and `InvalidOutputs`. The purpose of these functions will be discussed in the "Messaging" chapter.

### LoadDatabase

This call goes to [`state/loadDatabase.go`](https://github.com/FactomProject/factomd/blob/master/state/loadDatabase.go) and takes the node's State directly. This function call begins the process of loading the database. This will be discussed in the "Database" chapter.

### Timer

This call goes to [`Timer`](https://github.com/FactomProject/factomd/blob/master/engine/timer.go) and takes the node's State directly. It looks up the block time of of factomd (600 seconds on main net; same as BlkTime unless otherwise defined, such as in a test or simulator), does some math, and determines a `wait` and `next` value. These values are used in calls to `time.Sleep` inside the inner loop of this function. As the loop concludes, in adds a value between `[0,...,9]` to the `state.TickerQueue()` channel--which correspond approximately to "minutes" in the block. The size of the `AckQueue()` and `InMsgQueue()` affect the timing of this ticker.

### GoSyncEntries

This call runs on the fnode.State and can be found at [`state/entrySyncing.go`](https://github.com/FactomProject/factomd/blob/master/state/entrySyncing.go). `GoSyncEntries()` starts a messaging goroutine and runs a nonterminating loop to handle the work of discovering what entries or entry blocks are missing. When missing entries or entry blocks are found, this goroutine will submit messages to request missing entries and entry blocks. This part of the start up will be discussed in the "Database" Chapter.

### Elections Run/hackingfactom/

This call goes to/hackingfactom/lections/elections.go`](https://github.com/FactomProject/factomd/blob/master/elections/elections.go) and takes the node's State directly. This starts the main election loop for running elections. This will be discussed in the "Elections" chapter.

### ValidatorLoop/hackingfactom/

This call goes to [`ValidatorLoop`](https://github.com/FactomProject/factomd/blob/master/state/validation.go) and takes the node's State directly. If you take a peek at the Operations chapter, you'll see under the section about pprof that ValidatorLoop spends more time running than any other function and it triggers many other important, long running functions. ValidatorLoop is where we will go from this chapter into Messages.

[Next Chapter](/hackingfactom/messages){: .btn .btn-blue }

## State Reference

The [State Struct](https://github.com/FactomProject/factomd/blob/master/state/state.go) contains several properties that factomd initializes during start up and then continues to update during operation. A description of some of the noteworthy properties is found below.

| Name    | Type        | Description |
|:-------------|:------------------|:------|
| Salt          | IHash | A number generated by the node to tell if a message was originated by the node.  |
| IdentityChainID | IHash | If this node has an identity, this is it. Most nodes do not. |
| AuthorityServerCount | Int | Number of authority nodes allowed. |
| BootTime | Int64 | Time in seconds since node last booted |
| IgnoreDone | bool | If ignore mode has ended, this is a stop gap for performance reasons |
| IgnoreMissing | bool | Ignore missing messages during network reboot, where your own messages might show back up from the old network state |
| LLeaderHeight | unint32 | The current height -- the extra L makes it easier to look up in the IDE |
| EOM | bool | True when the first EOM is encountered (see glossary) |
| DBSig | bool | True when the first DBSig is encountered (see glossary) |
| Syncing | bool | True when looking for messages from leaders to sync |
| Holding | map[[32]byte]interfaces.IMsg | Follower holding messages |
| Acks | map[[32]byte]interfaces.IMsg | Following holding acknowledgements |
| HighestKnown | uint32 | Highest block for which we have received a message |
| EntryDBHeightComplete | uint32 | DBlock height this node has complete set of EBlocks and entries |
| EntryDBHeightProcessing | uint32 | DBlock Height at which we have started asking for or have all entries |
| DBFinished | bool | Database is loaded |
| DirectoryBlockInSeconds | int | Seconds per block, pulled from params for `BlkTime` |
| ProcessLists | *ProcessLists | Where temporary Factoid and Entry Credit balances are stored (discussed in "Messaging") |
| RunLeader | bool | If this node is allowed to run as a leader |
| Leader | bool | If this node is a leader |

## Additional reading

["factomd major process flow"](https://factomize.com/forums/threads/factomd-major-process-flow.1602/) by Who, February 7, 2019.

