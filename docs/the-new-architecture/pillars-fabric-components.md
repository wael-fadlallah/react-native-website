---
id: pillars-fabric-components
title: Fabric Components
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

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

### Folder Setup

The easiest way to create a Component is as a separate module we will then import as a dependency for our apps. This has several benefits, among which it pushes for code reuse, it keeps the components decoupled from the rest of the apps and it makes it easier to leverage some automation already in place.

For this guide, we are going to create a Fabric Component that centers some text on the screen.

Let's create a new folder at the same level of our app and let's call it `RTNCenteredText`.

In this folder, we are going to create three subfolders: `js`, `ios` and `android`.

The final result should look like this:

<figure>
  <img width="500" alt="Folder Structure for a Fabric Component" src="/docs/assets/NewArchitecture/AppFolderStructure.png"/>
  <figcaption>Initial folder structure for a Fabric Component.</figcaption>
</figure>

### JavaScript Specification

In the **New Architecture**, we decided to have a single source of truth for Components API and we decided that it has to be a typed dialect of JavaScript (either [Flow](https://flow.org/) or [TypeScript](https://www.typescriptlang.org/)). We need a typed dialect because the **Codegen** has to generate some code in C++, Objective-C++ and Java which are all strongly typed languages: therefore, it needs information on types.

Another important aspect of the JavaScript specification is the file name. A Component filename has to end with the `NativeComponent.js` (or `jsx`, `ts`, `tsx`) suffix. The **Codegen** process will look up for files ending with that suffix to generate the required code.

The following are the specification of our `RTNCenteredText` Component in both Flow and TypeScript.

<Tabs groupId="fabric-component-specs" defaultValue={constants.defaultJavaScriptSpecLanguages} values={constants.javaScriptSpecLanguages}>
<TabItem value="flow">

```typescript
// @flow strict-local

import type {ViewProps} from 'react-native/Libraries/Components/View/ViewPropTypes';
import type {HostComponent} from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

type NativeProps = $ReadOnly<{|
  ...ViewProps,
  text: ?string,
  // add other props here
|}>;

export default (codegenNativeComponent<NativeProps>(
   'RTNCenteredText',
): HostComponent<NativeProps>);
```

</TabItem>
<TabItem value="typescript">

```typescript
import type { ViewProps } from 'ViewPropTypes';
import type { HostComponent } from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

export interface NativeProps extends ViewProps {
  ...ViewProps,
  text: ?string,
  // add other props here
}

export default codegenNativeComponent<NativeProps>(
  'RTNCenteredText'
) as HostComponent<NativeProps>;
```

</TabItem>
</Tabs>

Let's break these down a little.

At the beginning of the spec files, there are the imports. Here, we can import what we need from React Native and other dependencies if needed. The most important imports, that are present in all the Fabric Components, are:

- The `HostComponent`, which represents what we have to export;
- The `codegenNativeComponent` function, which is responsible to actually register the Component in the JS runtime.

The second section of the files contains the **props** of the Component. Props are Component-specific information we want to be able to customize from the outside. In this case, we want to control the `text` the Component will render.

Finally, we invoke the `codegenNativeComponent` generic function, passing the name we want to use for our Component. The returned value is then exported by the JavaScript file in order to be used by the app.

### Component Configuration

### Native Code

#### iOS

#### Android

### Adding the Fabric Component To Your App
