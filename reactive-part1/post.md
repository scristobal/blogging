# Rubber duck series: Reactive programming in action
The rubber duck series is a series of blog posts, inspired by the [rubber duck debugging method](https://rubberduckdebugging.com/). 

Instead of a rubber duck, the target is an **advanced reader** already proficient with the topics covered. Basics will not be covered, there are much better resources out there. Instead, the focus will be on production-ready code with real examples and description of some of the architectural decisions. 

In this of post, the basic structure of some of [DataBeacon](www.databeacon.aero)’s software components will be covered. In particular, this post series shows how [reactive programming](https://reactivex.io/) is used in detail. 

> Code snippets have been adapted to this blog post specifically, it is not a 1:1 copy of production code and some implementation details have hidden. 
## Introduction
Victor5-funnel sits between Level5 [Kafka](https://kafka.apache.org/) topic and Victor5 [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA) clients. It coordinates client connections and transforms Kafka input stream into a web-socket connection (Socket.IO)  adapted to each client.

It is written in [TypeScript](https://www.typescriptlang.org/) and translated to [Node](https://nodejs.org/en/), although currently being migrated to [Go](https://go.dev/). This post focus on Node implementation, but I might cover the rationale and migration to Go in a future post. 

Let’s explore this component in detail. 
## Setting up clients
After setting up the environment details the main code starts with:

```typescript
const connection$ = await socketIOServer();
```
 
Here we set up a `connection$` observable of type `fromSocketIO` [link](www.mpn.js)  as new http request enter the server `connection$` will notify the subscribers with an object of type `ListenerAndSender`

```typescript
interface ListenerAndSender<L extends EventsMap, S extends EventsMap> {
    from: <Ev extends EventNames<L>>(eventName: Ev) => Observable<EventParam<L, Ev>[0]>;
    to: <Ev extends EventNames<S>>(eventName: Ev) => Observer<Parameters<S[Ev]>[0]>;
    id: string;
    user?: string;
    onDisconnect: (callback: () => void) => void;
}
```

This connection object can be used to monitor connections activity:

```typescript
connection$.subscribe(({ id, user, onDisconnect }) => {
        log(`Connected ${user} through session ${id}`);
        onDisconnect(() => {
            log(`Disconnected ${user} from session ${id}`);
        });
    });
```

This interface is used by the pure function `client` to generate an observable of `client$`

```typescript
const client$ = connection$.pipe(map((connector) => client(connector)));
```

Each `client` has an observable of `state$` that represents the always-changing client state. 

```typescript
client$.subscribe(({ state$ }) => state$.subscribe((state) => log(util.inspect(state, { depth: 4 }))));
```

Next thing we need is to attach a data source to the clients. Luckily, in addition to `state$` each client implements an `attachDataSource` and a `removeDataSource`

## Getting data from source

```typescript
const flight$ = await kafkaConnector();
```

We can monitor input data with a simple subscription: 

```typescript
flight$
	.subscribe(({flights}) => log(`Got a new msg containing ${flights.length} flights`));
```


Subscribe clients with

```typescript
    client$.subscribe((client) => client.attachDataSource(flight$));
```
 
## Filtering data 

Next chapter
