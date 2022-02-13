---
ics: TBA
title: Interchain Query
stage: draft
category: IBC/APP
requires: 25, 26, 27
kind: instantiation
author: Joe Schnetzler <schnetzlerjoe@gmail.com>
created: 2022-01-06
modified: 2022-02-12
---

## Synopsis

This documents aims to document the structure, plan and implementation of the Interchain Queries Module allowing for cross-chain querying of state from IBC enabled chains.

### Motivation

Interchain Accounts (ICS-27) brings one of the most important features IBC offers, cross chain transactions (on-chain). Limited in this functionality is the querying of state from one chain, on another chain. Adding interchain querying via the Interchain Query module, gives unlimited flexibility to chains to build IBC enabled protocols around Interchain Accounts and beyond.

### Definitions 

- `Querying Chain`: The chain that is interested in getting data from another chain (Queried Chain). The Querying Chain is the chain that implements the Interchain Query Module into their chain.
- `Queried Chain`: The chain that's state is being queried via the Querying Chain. The Queried Chain gets queried via a relayer utilizing its RPC client which is then submitted back to the Querying Chain.

### Desired Properties

- Permissionless: A Querying Chain can query a chain and implement cross-chain querying without any approval from a third party or chain governance.

- Minimal Querying Chain Work: A Queried Chain has to do no implementation work or add any module to enable cross chain querying. By utilizing an RPC client on a relayer, this is possible.

- Modular: Adding cross-chain querying should be as easy as implementing a module in your chain.

- Control Queried Data: The Querying Chain should have ultimate control on how to handle queried data.

- Incentivization: In order to incentivize relayers for participating in interchain queries, a bounty is paid.

## Technical Specification

### General Design 

The Querying Chain starts with the implementation of the Interchain Query Module by adding the module into their chain.

The general flow for interchain queries starts with a Cross Chain Query Request from the Querying Chain which is listened to by relayers. Upon recognition of a cross chain query, relayers utilize a ABCI Query Request to query data from the Queried Chain. Upon success, the relayer submits a `MsgSubmitQueryResult` to the Querying chain with the success flag as 1.

On failure of a query, relayers submit `MsgSubmitQueryResult` with the `success` flag as 0 to the Querying chain. Alternatively on timeout based on the height of the querying chain, the querying chain will submit `SubmitQueryTimeoutResult` with the timeout height specified.

### Data Structures

A CrossChainABCIQueryRequest data type is used to specify the query. Included in this is the.

```go
type CrossChainABCIQueryRequest struct {
	Path           string
	Key            string
	Id             string
	timeoutHeight  uint64
	Bounty         sdk.Coin
	ClientId       string
}
```

```go
type QueryResult struct {
	Data     []byte
	Key      string
	Id       string
	Height   uint64
	ClientId string
	Success  bool
}
```

```go
type QueryTimeoutResult struct {
	Key            string
	Id       	   string
	timeoutHeight  string
	ClientId       string
}
```

### Keepers

```go
func CrossChainABCIQueryRequest(
	
) {
  //Keeper to initiate interchain query request
}
```

```go
func SubmitQueryTimeoutResult(

) {
  //Keeper to submit a query timeout result. Applied at the beggining of each block and timedout when the querying chain hits timeout height.
}
```

### Messages

The querying chain has messages that the relayer can submit depending on a success/failed query.

```go
func MsgSubmitQueryResult(

) *cobra.Command {
  //Msg to submit a query result
}
```

The querying chain will have messages to get a list of interchain queries as well as get a query by type and id.

```go
func MsgGetQueries(

) *cobra.Command {
  //Msg to get a list of queries
}
```

```go
// Add -- max-height flag to allow for querying the highest height for a query (most recent)?
func MsgGetQuery(

) *cobra.Command {
  //Msg to get a query via id and key (i.e: id: 1 and key: OsmosisPool)
}
```

### Events

```go
func EmitQueryEvent(ctx sdk.Context, timeoutHeight exported.Height) {
	//Event to trigger query event on relayer
	ctx.EventManager().EmitEvents(sdk.Events{
		sdk.NewEvent(
		)
	})
}
```