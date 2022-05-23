---
id: pillars-fabric-components
title: Fabric Components
---

A Fabric Component is a UI component which is rendered on the screen using the [Fabric Renderer](https://reactnative.dev/architecture/fabric-renderer).

Using Fabric Components instead of Native Components allows us to reap all the [benefit](./why) coming from the **New Architecture**. Specifically, we are able to leverage JSI to efficiently connect the Native UI code and the JavaScript one, and all the other features provided by the new renderer.

A Fabric Component is created starting from a **JS specification**. This, with the help of a bit of [**Codegen**](./pillars-codegen), will create some C++ code, shared among all the RN's platforms. Then, after filling in the missing **native code** with Component specific logic, the Component can be **integrated** in the app.

The following section will guide you through the creation of a Fabric Component, step-by-step.

:::caution
Fabric Components only works with the **New Architecture** enabled.
To migrate to the **New Architecture**, follow the [Migration guide](../new-architecture-intro)
:::

## How to Create a Fabric Components

To create a Fabric Component, we have to follow these steps:

- Define a set of JS specification.
- Configure the Component so that it can be consumed by an app.
- Write the native code required to make it works.

Once these steps are done, the Component is ready to be consumend by an app. Therefore, the guide shows how to add it to an app, leveraging the _autolinking_, and how to reference it from the JavaScript code.

### Javascript Specification

### Component Configuration

### Native Code

#### iOS

#### Android

### Adding the Fabric Component To Your App
