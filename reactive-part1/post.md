# Reactive programming in action
This post shows how reactive programming is used under the hood in one of Level5 components. 

## Introduction

Victor5-funnel sits between Level5 Kafka topic and Victor5 clients. It coordinates client connections and transforms Kafka input stream into a web-socket connection (Socket.IO)  adapted to each client.

> Disclaimer: code snippets have been adapted to this blog post specifically, it is not a 1:1 copy of production code and some implementation details have hidden.  It you want more details on a particular topic, drop me a line at sc@databeacon.aero

## Setting up clients
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
connection$
	.subscribe(({ id, user, onDisconnect }) => {
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
client$
	.subscribe(({ state$ }) => state$.subscribe((state) => log(util.inspect(state, { depth: 4 }))));
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
