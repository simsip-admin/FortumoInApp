---
layout: home
title: Bill McRoberts
home: true
---

## Dependency Service for Carrier In-App Purchases (Fortumo)

Android Services &bull; Binding a Java Library &bull; Xamarin Forms &bull; DependencyService &bull; In-App Purchases &bull; Carrier Billing

Xamarin Forms support for cross-platform in-app purchasing via Fortumo for carrier billing.

<img src="images/screenshot-android-inapp.png" width="40%">

My DependencyService for In-App Purchases (IAP) is supported in all mobile markets in the world except Android for China - ouch. This may change in the future, but in the meantime, let's see if we can fill this gap. In researching this, I came across Fortumo ([http://fortumo.com](http://fortumo.com "Fortumo")), a service provider for carrier based billing with support for China. While they didn't have a Xamarin component, they did have an Android library. OK, I've done several Bindings to Java Libraries and I have a defined interface for an IAP DependencyService, let's see if we can implement a DependencyService for IAP via Fortumo's carrier billing service. The fact that Fortumo requires you to create an Android Service only makes this more interesting.

### Setup

### The Interface
The interface exposes an API and set of events to consume IAP.

> But wait you say, can't we do this with a Task based API instead of hooking up events? Hang in there that's exactly where I want to evolve this sample in the future.

Here is the API:

And here are the events:

### The Implementation

### The Sample App
I have kept the UI and MVVM architecture as simple as possible so that the focus can be on the transaction flows for IAP.

### Testing

### Publishing

   




