---
layout: page
title: Messages
permalink: /messages
nav_order: 3
---

# Messages
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Messages

As described in the introduction to this site, factomd is an application that connects to a network of other factomd nodes and trades information. Trading information between nodes is accomplished by using messages ([`IMsg`](https://github.com/FactomProject/factomd/blob/master/common/interfaces/msg.go), stop and read over it). Messages are processed through channels, and because of isolation between different goroutines, many messages are purely for internal use between different goroutines.

> Check the additional reading section for Who's post on messaging--it is a must-read and one you'll likely return to for reference while learning about factomd.
{: .fw-300}

During start up, [`engine/NetworkProcessorNet.go`](https://github.com/FactomProject/factomd/blob/master/engine/NetworkProcessorNet.go) is called. This starts a few goroutines which contain nonterminating loops. The parameter that `NetworkProcessorNet` takes is a `FactomNode`. And a `FactomNode` contains a list of [`IPeer`'s](https://github.com/FactomProject/factomd/blob/master/common/interfaces/peer.go). These peers are supplied by `P2PProxy` (discussed in the Peer to Peer chapter), which implements `IPeer`. In particular, it will implement `func (f *P2PProxy) Send(msg interfaces.IMsg)` and `func (f *P2PProxy) Receive() (interfaces.IMsg, error)`.

> Also during start up, inside [`NetStart.go`](https://github.com/FactomProject/factomd/blob/master/engine/NetStart.go), the P2PProxy is connected to the P2P Controller--creating the link between the factomd application and its lower level, isolated P2P package--effectively connecting the "cable" into factomd letting to reach out to find peers. Learn more in the Peer to Peer chapter.
{: .fw-300}

In the goroutine for `Peer` in `NetworkProcessorNet` with the nonterminating loop, there are functions and loops to do a large number of things for a single function (ignoring things while syncing, ignoring duplicates, etc.), but one part we'll consider is the [`loop through the Peers`](https://github.com/FactomProject/factomd/blob/master/engine/NetworkProcessorNet.go#L194) list to use the `Receive` method.

Upon using the `Receive` method, the application has taken in broadcasts from its peer. The message and any of its errors pass through a large number of validation steps and re-broadcast tagging. Once validated, the message will pass into one of two queues. If the message type is any of [[`REVEAL_ENTRY_MSG`, `COMMIT_CHAIN_MSG`, `COMMIT_ENTRY_MSG`]](https://github.com/FactomProject/factomd/blob/master/common/constants/constants.go#L88) then it is placed into `State.InMsgQueue2()`, otherwise `State.InMsgQueue()`.

### Messages as Data Structures

Check Who's post on messaging--["Explanation of Messages (IMsg)"](https://factomize.com/forums/threads/explanation-of-messages-imsg.1724/)--it is a must-read and one you'll likely return to for reference.

## What Messages Do

Messages do not always merely contain static pieces of data, but also trigger state-altering events. The information in messages helps the node figure out a wide range of actions to take, such as setting an election timeout or how to order messages and process the VMs. 

Thus it is helpful to think of messages as being something that you execute.
{: .fw-500}

Let's turn to how factomd handles building blocks and the central role that messages play. The first step toward that end is looking at when the ValidatorLoop calls on Process().

[Next Chapter](/hackingfactom/elections){: .btn .btn-blue }

## Additional Reading

["Explanation of Messages (IMsg)"](https://factomize.com/forums/threads/explanation-of-messages-imsg.1724/) by Who, March 4, 2019.

["Factomd P2P/Network flow diagram"](https://factomize.com/forums/threads/factomd-p2p-network-flow-diagram.1653/) by Who, February 17, 2019.