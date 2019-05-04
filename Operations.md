---
layout: page
title: Operations
permalink: /operations/
nav_order: 6
---

# Operations
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Profiling factomd with pprof

pprof is a tool used by Go developers to see where resources are used by a Go program. Factomd has pprof available for you to pull samples.

A couple example usages of pprof include getting a look at the heap profile or polling the program periodically to pull in samples. If you run pprof for a few minutes you might end up with hundreds of samples.

With those samples you can do neat things like get an SVG representation of how factomd is spending time and space. pprof is an invaluable tool for diagnosing high CPU or high memory usage, as well as for general optimization.

Here is an image example of pprof output for factomd:

![pprof](/hackingfactom/assets/pprof.png "pprof")

Factomd runs a pprof HTTP server at port `6060` and you can pull a 30 second profile with:

`go tool pprof http://<FACTOM_HOST>:6060/debug/pprof/profile`

For information on how to use pprof, as well as grabbing different profiles, check out [pprof](https://golang.org/pkg/runtime/pprof/) and [Profiling Go Programs](https://blog.golang.org/profiling-go-programs).

## HTTP API

factomd runs an API served over HTTP. The code that operates the web server is under `/wsapi`.

The API is versioned with two versions currently available. The API paths are prefixed by `/v1` or `/v2` to distinguish version.

V1 paths start around [https://github.com/FactomProject/factomd/blob/master/wsapi/wsapi.go#L63](https://github.com/FactomProject/factomd/blob/master/wsapi/wsapi.go#L63).

```go
server.Post("/v1/factoid-submit/?", HandleFactoidSubmit)
server.Post("/v1/commit-chain/?", HandleCommitChain)
server.Post("/v1/reveal-chain/?", HandleRevealChain)
server.Post("/v1/commit-entry/?", HandleCommitEntry)
server.Post("/v1/reveal-entry/?", HandleRevealEntry)
server.Get("/v1/directory-block-head/?", HandleDirectoryBlockHead)
server.Get("/v1/get-raw-data/([^/]+)", HandleGetRaw)
server.Get("/v1/get-receipt/([^/]+)", HandleGetReceipt)
server.Get("/v1/directory-block-by-keymr/([^/]+)", HandleDirectoryBlock)
server.Get("/v1/directory-block-height/?", HandleDirectoryBlockHeight)
server.Get("/v1/entry-block-by-keymr/([^/]+)", HandleEntryBlock)
server.Get("/v1/entry-by-hash/([^/]+)", HandleEntry)
server.Get("/v1/chain-head/([^/]+)", HandleChainHead)
server.Get("/v1/entry-credit-balance/([^/]+)", HandleEntryCreditBalance)
server.Get("/v1/factoid-balance/([^/]+)", HandleFactoidBalance)
server.Get("/v1/factoid-get-fee/", HandleGetFee)
server.Get("/v1/properties/", HandleProperties)
server.Get("/v1/heights/", HandleHeights)
server.Get("/v1/dblock-by-height/([^/]+)", HandleDBlockByHeight)
server.Get("/v1/ecblock-by-height/([^/]+)", HandleECBlockByHeight)
server.Get("/v1/fblock-by-height/([^/]+)", HandleFBlockByHeight)
server.Get("/v1/ablock-by-height/([^/]+)", HandleABlockByHeight)
```

V2 API paths are handled separately inside the [wsapiV2.go](https://github.com/FactomProject/factomd/blob/master/wsapi/wsapiV2.go) file. V2 behaves more like a JSON RPC API than a simpler REST API. You send a request along with a payload that requests a method to execute.

The methods available for execution are listed inside a switch statement inside [HandleV2Request()](https://github.com/FactomProject/factomd/blob/master/wsapi/wsapiV2.go#L71).

```go
	switch j.Method {
	case "chain-head":
		resp, jsonError = HandleV2ChainHead(state, params)
		break
	case "commit-chain":
		resp, jsonError = HandleV2CommitChain(state, params)
		break
	case "commit-entry":
		resp, jsonError = HandleV2CommitEntry(state, params)
		break
	case "current-minute":
		resp, jsonError = HandleV2CurrentMinute(state, params)

```

*Truncated for length.*

In V2 methods are passed to handler functions. Note that the handler functions presently draw information from the factomd IState interface and thus perform a mutex lock and unlock when grabbing a copy of state. This is handled once in V2, but V1 endpoints perform this lock inside each handler.

## Prometheus

factomd exports data to be consumed by Prometheus. [Prometheus](https://prometheus.io/) is an open source monitoring tool that operates a central server that polls servers it is monitoring for exports of system and application data. Example:

```
# HELP factomd_StartingPoint_peers_broadcast Number of msgs broadcasting
# TYPE factomd_StartingPoint_peers_broadcast gauge
factomd_StartingPoint_peers_broadcast 0
# HELP factomd_database_leveldb_cacheblock Memory used by Level DB for caching
# TYPE factomd_database_leveldb_cacheblock gauge
factomd_database_leveldb_cacheblock 0
# HELP factomd_database_leveldb_gets Counts gets from the database
# TYPE factomd_database_leveldb_gets counter
factomd_database_leveldb_gets 158388
```

You can start a Prometheus server to target your factom node on port 9876. When you are running a node, visit `http://<FACTOM_HOST>:9876/metrics` to see some the export. Examples of how to set up a can be found in the factomd repository under `support` directory. In particular, this [prometheus.yml](https://github.com/FactomProject/factomd/blob/master/support/dev/prometheus/config/prometheus.yml).

