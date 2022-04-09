---
description: An further example of reactive programing, using Kafka, Socket.IO and TypeScript
tags: RxJS, typescript, kafka, socketio, nodejs
license: public-domain
published: true
---

# Reactive programming in action - part 2

This post is the continuation of a previous post on how [reactive programming](https://reactivex.io/) is used in one of [DataBeacon](www.databeacon.aero)’s central software component, called _Funnel_. The post is inspired by the [rubber duck debugging method](https://rubberduckdebugging.com/).

Instead of covering the basics, there are much better resources out there, the focus will be on production-ready code with real examples and description of some of the architectural decisions.

> Code snippets have been adapted to this blog post specifically, it is not a 1:1 copy of production code and some implementation details have hidden.

## Introduction

## Creating the clients

Setup the client function, the `state$` observable is straight forward from `rxkfk`

```typescript
const client = ({ from, to }: Connector<ListenEventMap, SendEventMap>) => {
    const state$ = from('state');

    function attachDataSource(serverMessage$: BehaviorSubject<TimeSeriesItem>) {}

    function removeDataSource() {}

    return Object.freeze({ state$, attachDataSource, removeDataSource });
};
```

The `attachDataSource` is done in three steps:

1.  build the `emitter` observer, which is straight forward from `rxkfk`

2.  create the `flight$` observable source, `clientProjection` is basically a `message` filter that uses `clientState` to determine if the message should be forwarded. The signature is

    ```typescript
    const clientProjection = (serverState: TimeSeriesItem, clientState: ClientState) => TimeSeriesItem;
    ```

3.  finally subscribe the `emitter` observer to the `flight$` observable.

```typescript
function attachDataSource(serverMessage$: BehaviorSubject<TimeSeriesItem>) {
    const emitter = to('data');

    const flight$ = combineLatest([serverMessage$, state$]).pipe(
        map(([message, clientState]) => clientProjection(message, clientState)),
        filter((flights) => flights !== undefined)
    );

    flight$.subscribe(emitter);
}
```

Last, to implement `removeDataSource` we need to create a closure that captures the `subscription`. Once the `subscription` is available, the `removeDataSource` is trivial.

```typescript
const client = ({ from, to }: Connector<ListenEventMap, SendEventMap>) => {
    const subscription = new Subscription();

    const state$ = from('state');

    function attachDataSource(serverMessage$: BehaviorSubject<TimeSeriesItem>) {
        const emitter = to('data');

        const flight$ = combineLatest([serverMessage$, state$]).pipe(
            map(([message, clientState]) => clientProjection(message, clientState)),
            filter((flights) => flights !== undefined)
        );

        subscription.add(flight$.subscribe(emitter));
    }

    function removeDataSource() {
        subscription?.unsubscribe();
    }

    return Object.freeze({ state$, attachDataSource, removeDataSource });
};
```
