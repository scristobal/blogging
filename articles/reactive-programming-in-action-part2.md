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
