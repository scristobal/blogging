---
description: An example of reactive programing, using Kafka, Socket.IO and TypeScript
tags: RxJS, typescript, kafka, socketio, nodejs
license: public-domain
published: false
---

# Reactive programming in action - part 1

This post shows how [reactive programming](https://reactivex.io/) is used in one of [DataBeacon](www.databeacon.aero)’s central software component, called _Funnel_. The post is inspired by the [rubber duck debugging method](https://rubberduckdebugging.com/).

Instead of covering the basics, there are much better resources out there, the focus will be on production-ready code with real examples and description of some of the architectural decisions.

> Code snippets have been adapted to this blog post specifically, it is not a 1:1 copy of production code and some implementation details have hidden.

## Introduction

The _Funnel_ component sits between the [Kafka](https://kafka.apache.org/) topics and the [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA) clients. It coordinates client connections and transforms a Kafka input stream into a web-socket connection (Socket.IO) adapted to each client status and preferences.

Kafka topics -> Funnel -> clients

_Funnel_ is written in [TypeScript](https://www.typescriptlang.org/) and translated to [Node](https://nodejs.org/en/), although currently being migrated to [Go](https://go.dev/). This post focus on Node implementation, but I might cover the rationale and migration to Go in a future post.

Let’s explore _Funnel_ in detail.

## Setting up clients

After setting up the environment details the main code starts with:

```typescript
const connection$ = await socketIOServer();
```

Here we create up a `connection$` observable of type `fromSocketIO` using [rxjs-socket.io](https://github.com/scristobal/rxjs-socket.io). For each new http request, `connection$` will notify the subscribers with an object of type `Connector`

```typescript
interface Connector<L extends EventsMap, S extends EventsMap> {
    from: <Ev extends EventNames<L>>(eventName: Ev) => Observable<EventParam<L, Ev>>;
    to: <Ev extends EventNames<S>>(eventName: Ev) => Observer<Parameters<S[Ev]>>;
    id: string;
    user?: string;
    onDisconnect: (callback: () => void) => void;
}
```

Note the first two methods, they are both a factory:

-   `from` takes an event name as parameter and produces an `Observable` from [receive](https://socket.io/docs/v4/server-api/#socketoneventname-callback).

-   `to` takes an event name as a parameter and produces an `Observer` to [emit](https://socket.io/docs/v4/server-api/#socketemiteventname-args).

This allows things like `from('action').subscribe(to('reducer'))` which could be used to manage client state remotely.

The parameters `id` and `user` are self-descriptive and `onDisconnect` registers a callback that will be executed upon the client’s disconnection.

Under the hood, the Socket.IO server is protected by the [auth0-socketio](https://www.npmjs.com/package/auth0-socketio) middleware to mange authentication with the Auth0 identity provider.

This connection object can be used to monitor connections activity:

```typescript
connection$.subscribe(({ id, user, onDisconnect }) => {
    log(`Connected ${user} through session ${id}`);
    onDisconnect(() => log(`Disconnected ${user} from session ${id}`));
});
```

This interface is used by the function `client` to generate an observable of `client$`

```typescript
const client$ = connection$.pipe(map((connector) => client(connector)));
```

Each `client` has an observable of `state$` that updates with the client state, which is implemented using React+Redux.

```typescript
client$.subscribe(({ state$ }) => state$.subscribe((state) => log(util.inspect(state, { depth: 4 }))));
```

Next thing we need is to attach a data source to the clients. Luckily, in addition to `state$` each client implements an `attachDataSource` and a `removeDataSource`

Only one source can be attached at a time, `attachDataSource` expects an observable and `removeDataSource` is just a function that unsubscribes the client from the source updates.

That’s all we need for now, we will describe the `client` generation in a future post. Let’s now setup the data sources.

## Getting data from source

To transform a Kafka topic to an observable we use the [rxkfk](https://www.npmjs.com/package/rxjs-kafka) library. Details of the connection are hidden inside the `kafkaConnector` but it returns a typed observable with the messages of a given topic(s).

```typescript
const msg$ = await kafkaConnector();
```

We can monitor input data with a simple subscription:

```typescript
msg$.subscribe(({ flights }) => log(`Got a new msg containing ${flights.length} flights`));
```

Finally, each client should be subscribed individually. In case we want to attach all clients to the same source we should simply attach an observer to `client$` to establish the link between each individual `client` and the `msg$` observable.

```typescript
client$.subscribe((client) => client.attachDataSource(msg$));
```

In addition to attach data sources, clients can unsubscribe calling the `client.removeDataSource()` method. This allows clients to dynamically change data sources.

## Coming next

So far we have covered the basic structure of the code: created two observables for client and server side and _programmagically_ ✨ connected both.

In then next chapter we will fill the gaps and describe how clients and data sources are connected, how to create `clients` from `connections` and how data sources are filtered using a [projection and combination operator](https://rxjs.dev/api/index/function/combineLatest).
