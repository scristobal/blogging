# Reactive programming in action
This post is inspired by the [rubber duck debugging method](https://rubberduckdebugging.com/). But instead of a rubber duck, the target is an **advanced reader** already proficient with the topics covered. Basics will not be covered, there are much better resources out there. Instead, the focus will be on production-ready code with real examples and description of some of the architectural decisions. 

In particular, this post shows how [reactive programming](https://reactivex.io/) is used in one of [DataBeacon](www.databeacon.aero)’s central software component, called *Funnel*.

> Code snippets have been adapted to this blog post specifically, it is not a 1:1 copy of production code and some implementation details have hidden. 
## Introduction
The *Funnel* component sits between the  [Kafka](https://kafka.apache.org/) topics and the [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA) clients. It coordinates client connections and transforms a Kafka input stream into a web-socket connection (Socket.IO)  adapted to each client status and preferences. 

Kafka topics -> Funnel -> clients

*Funnel* is written in [TypeScript](https://www.typescriptlang.org/) and translated to [Node](https://nodejs.org/en/), although currently being migrated to [Go](https://go.dev/). This post focus on Node implementation, but I might cover the rationale and migration to Go in a future post. 

Let’s explore *Funnel* in detail. 
## Setting up clients
After setting up the environment details the main code starts with:

```typescript
const connection$ = await socketIOServer();
```
 
Here we create up a `connection$` observable of type `fromSocketIO` [link](www.mpn.js), as new http requests enter the server, `connection$` will notify the subscribers with an object of type `Connector`

```typescript
interface Connector<L extends EventsMap, S extends EventsMap> {
	from: <Ev extends EventNames<L>>(eventName: Ev) => Observable<EventParam<L, Ev>>;
	to: <Ev extends EventNames<S>>(eventName: Ev) => Observer<Parameters<S[Ev]>>;
	id: string;
	user?: string;
	onDisconnect: (callback: () => void) => void;
}
```

Note the `from` and `to` methods, they both return a factory of `Observables` parametrised by event names to [receive](https://socket.io/docs/v4/server-api/#socketoneventname-callback) or [emit](https://socket.io/docs/v4/server-api/#socketemiteventname-args) data respectively. 

This connection object can be used to monitor connections activity:

```typescript
connection$.subscribe(({ id, user, onDisconnect }) => {
	log(`Connected ${user} through session ${id}`);
	onDisconnect(() => log(`Disconnected ${user} from session ${id}`));
});
```

This interface is used by the pure function `client` to generate an observable of `client$`

```typescript
const client$ = connection$.pipe(map((connector) => client(connector)));
```

Each `client` has an observable of `state$` that represents updates in the client state. 

```typescript
client$.subscribe(({ state$ }) => state$.subscribe((state) => log(util.inspect(state, { depth: 4 }))));
```

Next thing we need is to attach a data source to the clients. Luckily, in addition to `state$` each client implements an `attachDataSource` and a `removeDataSource` 

Only one source can be attached at a time, `attachDataSource` expects an observable and `removeDataSource` is just a function that unsubscribes the client from the source updates. 

That’s all we need for now, we will describe the `client` generation in a future post.  Let’s now setup the data sources. 
## Getting data from source


```typescript
const flight$ = await kafkaConnector();
```

We can monitor input data with a simple subscription: 

```typescript
flight$.subscribe(({flights}) => log(`Got a new msg containing ${flights.length} flights`));
```


Subscribe clients with

```typescript
client$.subscribe((client) => client.attachDataSource(flight$));
```
 
## Coming next

So far we have covered the basic structure of the code: created two observables for client and server side and *programmagically* connected both. 

In then next chapter we will fill the gaps and describe how clients and data sources are connected, how to create `clients` from `connections` and how data sources are filtered using a [projection and combination operator](https://rxjs.dev/api/index/function/combineLatest). 