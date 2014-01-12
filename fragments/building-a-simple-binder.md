---
title: Building a Simple Binder
---

Data binders appear to be quite complex but this is mostly due to all the convenience features they come with. At their core, data binders are nothing more than a simple PubSub pattern.

A data-binding system can be boiled down to three elements

1. A way of specifying which UI elemenets are bound to which propertiers
2. A way to monitor changes both on the UI and on the properties
3. A way to trigger/alert both the UI and the object of any changes
