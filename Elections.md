---
layout: page
title: Elections
permalink: /elections/
nav_order: 4
exclude: true
---

# Elections
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Run

Elections help factomd determine the set of servers that will belong in the Federated Server Pool. As discussed in the Factom Whitepaper, if a Federated Server faults, an Audit server will take its place. The process of elections makes this happen.

`Run()` creates a new [`Election`](https://github.com/FactomProject/factomd/blob/master/elections/elections.go) and connects the `ElectionQueue` from State into the `Election`. It starts a nonterminating loop and calls [`BlockingDequeue`](https://github.com/FactomProject/factomd/blob/master/state/specialQueues.go#L87) on the aforementioned `ElectionQueue`. This will block until an election message arrives. It will be an [`IMsg`](https://github.com/FactomProject/factomd/blob/master/common/interfaces/msg.go) asserted to be an [`IElectionMsg`](https://github.com/FactomProject/factomd/blob/master/common/interfaces/elections.go).

### The Election Message

The Election message has two election-specific functions: `ElectionProcess` and `ElectionValidate`. Different [`types of elections messages`](https://github.com/FactomProject/factomd/tree/master/common/messages/electionMsgs) will execute different versions of those two messages. 

To see an inventory of types of elections messages and their origins, see the "Election Messages" post of Who's ["Explanation of Messages (IMsg)"](https://factomize.com/forums/threads/explanation-of-messages-imsg.1724/#post-13324) thread. You can see in the directory linked in the last paragraph the validation and process functions that are called.

[Next Section](/hackingfactom/peer-to-peer){: .btn .btn-blue }