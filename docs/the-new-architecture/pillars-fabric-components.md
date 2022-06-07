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

## 1. Folder Setup

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

## 2. Component Configuration

The second element we need to properly develop the Component is a bit of configuration, that will help you setting up:

- the Codegen
- the files to link the Component in the app.

Some of these configuration are shared between iOS and Android, while the others are platform specific.

### Shared

The shared bit is a `package.json` file that will be used by yarn to install your Component.
The file shall live at the root of your Component, side-by-side to the folders created at the [folder Setup](#folder-setup) step.

```json
{
  "name": "rtn-centered-text",
  "version": "0.0.1",
  "description": "Showcase a Fabric Component with a centered text",
  "react-native": "js/index",
  "source": "js/index",
  "files": [
    "js",
    "android",
    "ios",
    "rtn-centered-text.podspec",
    "!android/build",
    "!ios/build",
    "!**/__tests__",
    "!**/__fixtures__",
    "!**/__mocks__"
  ],
  "keywords": ["react-native", "ios", "android"],
  "repository": "https://github.com/<your_github_handle>/rtn-centered-text",
  "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<your_github_handle>/rtn-centered-text/issues"
  },
  "homepage": "https://github.com/<your_github_handle>/rtn-centered-text#readme",
  "devDependencies": {},
  "peerDependencies": {
    "react": "*",
    "react-native": "*"
  },
  // highlight-start
  "codegenConfig": {
    "name": "RTNCenteredTextSpecs",
    "type": "all",
    "jsSrcsDir": "js",
    "android": {
      "javaPackageName": "com.RTNCenteredText"
    }
  }
  // highlight-end
}
```

Let's break it down.

The upper part of the file contains some descriptive information like the name of the Component, its version and what composes it.
Make sure to update the various placeholders which are wrapped in `<>`, so replace all the occurrences of the `<your_github_handle>`, `<Your Name>`, and `<your_email@your_provider.com>` tokens.

Then we have the dependencies for this package, specifically we need `react` and `react-native`, but you can add all the other dependencies you may have.

Finally, the last important bit is the `codegenConfig` tag. This will be used **In Development** to automatically generate the code. It will be useful to see what is generated so that we can refer to the proper types. This object contains three keys:

- `name`: The name of the library we are going to generate. By convention, we add the `Specs` suffix.
- `type`: You can choose between `components` and `all`. Our suggestion is to leave `all` because it will push for extensibility in case you'll want to add also a TurboModule to this Component.
- `jsSrcsDir`: the relative path to access the `js` specification that will be parsed by the Codegen.
- `android`: an object that contains android-specific settings.
  - `javaPackageName`: the name of the package that must be used to generate the code.

### iOS - Create the `podspec` File

Now it's time to configure the native Component so that we can generate the required code and we can include it in our app.

For iOS, we need to create a `podspec` file which will define the Component as a dependency.
The `podspec` file for our Component will look like this

```ruby title="rtn-centered-text.podspec"
require "json"

package = JSON.parse(File.read(File.join(__dir__, "package.json")))

folly_version = '2021.06.28.00-v2'
folly_compiler_flags = '-DFOLLY_NO_CONFIG -DFOLLY_MOBILE=1 -DFOLLY_USE_LIBCPP=1 -Wno-comma -Wno-shorten-64-to-32'

Pod::Spec.new do |s|
  s.name            = "rtn-centered-text"
  s.version         = package["version"]
  s.summary         = package["description"]
  s.description     = package["description"]
  s.homepage        = package["homepage"]
  s.license         = package["license"]
  s.platforms       = { :ios => "11.0" }
  s.author          = package["author"]
  s.source          = { :git => package["repository"], :tag => "#{s.version}" }

  s.source_files    = "ios/**/*.{h,m,mm,swift}"

  s.dependency "React-Core"

  s.compiler_flags = folly_compiler_flags + " -DRCT_NEW_ARCH_ENABLED=1"
  s.pod_target_xcconfig    = {
    "HEADER_SEARCH_PATHS" => "\"$(PODS_ROOT)/boost\"",
    "OTHER_CPLUSPLUSFLAGS" => "-DFOLLY_NO_CONFIG -DFOLLY_MOBILE=1 -DFOLLY_USE_LIBCPP=1",
    "CLANG_CXX_LANGUAGE_STANDARD" => "c++17"
  }

  s.dependency "React-RCTFabric"
  s.dependency "React-Codegen"
  s.dependency "RCT-Folly", folly_version
  s.dependency "RCTRequired"
  s.dependency "RCTTypeSafety"
  s.dependency "ReactCommon/turbomodule/core"
end
```

The `podspec` file has to be a sibling of the `package.json` file and its name is the one we set in the `package.json`'s `name` property: `rtn-centered-text`.

The first part of the file prepares some variables we will use throughout the rest of it:

- the `package` variable which contains information read from the `package.json`
- the `folly_version` variable to set the version of `folly` we need.
- the `folly compiler_flags` that we need to set in the new architecture.

The next section contains some information used to configure the pod, like its name, version, a description and so on.

Finally, we have a set of dependencies that are required by the new architecture.

### Android - Create the `build.gradle` File

For what concerns Android, we need to create a `build.gradle` file in the `android` folder. The file will have the following shape

```ruby title="build.gradle"
buildscript {
    ext.safeExtGet = {prop, fallback ->
        rootProject.ext.has(prop) ? rootProject.ext.get(prop) : fallback
    }
    repositories {
        google()
        gradlePluginPortal()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:7.1.1")
    }
}

apply plugin: 'com.android.library'
apply plugin: 'com.facebook.react'

android {
    compileSdkVersion safeExtGet('compileSdkVersion', 31)

    defaultConfig {
        minSdkVersion safeExtGet('minSdkVersion', 21)
        targetSdkVersion safeExtGet('targetSdkVersion', 31)
        buildConfigField("boolean", "IS_NEW_ARCHITECTURE_ENABLED", "true")
    }
}

repositories {
    maven {
        // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
        url "$projectDir/../node_modules/react-native/android"
    }
    mavenCentral()
    google()
}

dependencies {
    implementation 'com.facebook.react:react-native:+'
}
```

## 3. Native Code

The last step requires us to write some native code to connect the JS side of our Component to what is offered by the platforms. This process requires two main steps:

- Run the **CodeGen** to see what would be generated
- Write the native code that will make it work

:::caution
The code generated by the **CodeGen** in this step should not be committed to the versioning system. React Native apps are able to generate
the code when the app is built. This allows to avoid any ABI incompatibility and to ensure that a consistent version of the codege is used.
:::

### iOS - CodeGen, View and ViewManager

#### Generate the code - iOS

To generate the code starting from the JS specs for iOS, we need to open a terminal and run the following command:

```sh
cd MyApp
yarn add ../RTNCenteredText
cd ..
node MyApp/node_modules/react-native/scripts/generate-artifacts.js \
  --path MyApp/ \
  --outputPath RTNCenteredText/generated/
```

This script first makes our Component visible to the app, so that we can generate the **CodeGen** from it, using `yarn add` to add a local version of the NPM package.

Then, it uses `node` to invoke the `generate-artifacts.js` script, which is the responsible to generate the code for iOS. It requires us to pass the path to a React Native app which contains our Component, using the `--path` parameter.

Then, we specify the `--outputPath` where we want it to generate the code.

The output of this process is the following folder structure:

```sh
generated
└── build
    └── generated
        └── ios
            ├── FBReactNativeSpec
            │   ├── FBReactNativeSpec-generated.mm
            │   └── FBReactNativeSpec.h
            ├── RCTThirdPartyFabricComponentsProvider.h
            ├── RCTThirdPartyFabricComponentsProvider.mm
            └── react
                └── renderer
                    └── components
                        ├── RTNCenteredTextSpecs
                        │   ├── ComponentDescriptors.h
                        │   ├── EventEmitters.cpp
                        │   ├── EventEmitters.h
                        │   ├── Props.cpp
                        │   ├── Props.h
                        │   ├── RCTComponentViewHelpers.h
                        │   ├── ShadowNodes.cpp
                        │   └── ShadowNodes.h
                        └── rncore
                            ├── ComponentDescriptors.h
                            ├── EventEmitters.cpp
                            ├── EventEmitters.h
                            ├── Props.cpp
                            ├── Props.h
                            ├── RCTComponentViewHelpers.h
                            ├── ShadowNodes.cpp
                            └── ShadowNodes.h
```

The relevant path for the Component we are writing is `generated/build/generated/ios/react/renderer/components/RTNCenteredTextSpecs`.
This folder contains all the genereted code required by our Component.

See the [CodeGen](./pillars-codegen) section for further details on the generated files.

#### Write the Native iOS Code

Now that we can see the iOS code we need, it's time to write the Native code for our Fabric Component.
We need to create three files:

1. The `RTNCenteredTextManager.mm`, an Objective-C++ file which declares what the Component exports.
2. The `RTNCenteredText.h`, an header file for the actual view.
3. The `RTNCenteredText.mm`, the implementation of the view.

##### RTNCenteredTextManager.mm

```obj-c title="RTNCenteredTextManager.mm"
#import <React/RCTLog.h>
#import <React/RCTUIManager.h>
#import <React/RCTViewManager.h>

@interface RTNCenteredTextManager : RCTViewManager
@end

@implementation RTNCenteredTextManager

RCT_EXPORT_MODULE(RTNCenteredText)

RCT_EXPORT_VIEW_PROPERTY(text, NSString)

- (UIView *)view
{
  return [[UIView alloc] init];
}

@end
```

This file is the manager for the Fabric Component.

The most important call is to the `RCT_EXPORT_MODULE` which is required to export the module so that fabric can actually retrieve it and instantiate it.

Then, we have to expose a property for the Fabric Component. This is done with the `RCT_EXPORT_VIEW_PROPERTY` macro, specifieng a name and a type.

Finally, we need to add a `- ((UIView *)view)` method for legacy reasons.

:::info
There are other macros that can be used to export custom properties, emitters and other constructs. You can look them up [here](https://github.com/facebook/react-native/blob/main/React/Views/RCTViewManager.h)
:::

##### RTNCenteredText.h

```Objective-C title="RTNCenteredText.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface RTNCenteredText : RCTViewComponentView

@end

NS_ASSUME_NONNULL_END
```

This file defines the interface for the `RTNCenteredText` view. Here, we can add any native method we may want to invoke on the view. For this guide, we don't need anything, therefore the interface is empty.

##### RTNCenteredText.mm

```C++ title="RTNCenteredText.mm"
#import "RTNCenteredText.h"

#import <react/renderer/components/RTNCenteredTextSpecs/ComponentDescriptors.h>
#import <react/renderer/components/RTNCenteredTextSpecs/EventEmitters.h>
#import <react/renderer/components/RTNCenteredTextSpecs/Props.h>
#import <react/renderer/components/RTNCenteredTextSpecs/RCTComponentViewHelpers.h>

#import "RCTFabricComponentsPlugins.h"

using namespace facebook::react;

@interface RTNCenteredText () <RCTRTNCenteredTextViewProtocol>
@end

@implementation RTNCenteredText {
  UIView *_view;
  UILabel *_label;
}

+ (ComponentDescriptorProvider)componentDescriptorProvider
{
  return concreteComponentDescriptorProvider<RTNCenteredTextComponentDescriptor>();
}

- (instancetype)initWithFrame:(CGRect)frame
{
  if (self = [super initWithFrame:frame]) {
    static const auto defaultProps = std::make_shared<const RTNCenteredTextProps>();
    _props = defaultProps;

    _view = [[UIView alloc] init];
    _view.backgroundColor = [UIColor redColor];

    _label = [[UILabel alloc] init];
    _label.text = @"Initial value";
    [_view addSubview:_label];

    _label.translatesAutoresizingMaskIntoConstraints = false;
    [NSLayoutConstraint activateConstraints:@[
      [_label.leadingAnchor constraintEqualToAnchor:_view.leadingAnchor],
      [_label.topAnchor constraintEqualToAnchor:_view.topAnchor],
      [_label.trailingAnchor constraintEqualToAnchor:_view.trailingAnchor],
      [_label.bottomAnchor constraintEqualToAnchor:_view.bottomAnchor],
    ]];

    _label.textAlignment = NSTextAlignmentCenter;

    self.contentView = _view;
  }

  return self;
}

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps
{
  const auto &oldViewProps = *std::static_pointer_cast<RTNCenteredTextProps const>(_props);
  const auto &newViewProps = *std::static_pointer_cast<RTNCenteredTextProps const>(props);

  if (oldViewProps.text != newViewProps.text) {
    _label.text = [[NSString alloc] initWithCString:newViewProps.text.c_str() encoding:NSASCIIStringEncoding];
  }

  [super updateProps:props oldProps:oldProps];
}

@end

Class<RCTComponentViewProtocol> RTNCenteredTextCls(void)
{
  return RTNCenteredText.class;
}
```

This file contains the actual implementation of the view.

It starts with some import which requires us to read from the files generated by the **CodeGen**.

It has to conform to a specific protocol, in this case `RCTRTNCenteredTextViewProtocol`, which is generated by the **CodeGen**.

It defines a static `(ComponentDescriptorProvider)componentDescriptorProvider` method, which is used by Fabric to retrieve the Descriptor provider to instantiate the object.

Then, we can initialize the view as we usually do with iOS views. In the `init` method, it is important to create a `defaultProps` struct using the `RTNCenteredTextProps` type from the **CodeGen**. We need to assign it to the private `_props` property to correctly initialize the Fabric Component. The remaining part of the initializer is standard Objective-C code to create views and layout them with AutoLayout.

The last two pieces are the `updateProps` method and the `RTNCenteredTextCls` method.

The `updateProps` method is invoked by Fabric every time a prop changes in JS. We can then cast the props passed as parameters to the proper `RTNCenteredTextProps` type and update the native code if it needs to be updated.

Notice that the superclass method `[super updateProps]` must be invoked as the last statement of this method, otherwise the `props` and `oldProps` struct will have the same values.

Finally, the `RTNCenteredTextCls` is another static method used to retrieve the correct instance of the class at runtime.

:::caution
Differently from Native Components, Fabric requires us to manually implement the `updateProps` method. It's not enough to export properties with the `RCT_EXPORT_XXX` and `RCT_REMAP_XXX` macros.
:::

### Android - CodeGen, View and ViewManager

Android follows some similar steps to iOS. We have to generate the code for Android, and then we have to write some native code to make it works.

#### Generate the Code - Android

To generate the code for Android, we need to manually invoke the CodeGen. This is done similarly to what we did for iOS: first, we need to add the package to the app and then we need to invoke a script.

```sh title="Running CodeGen for Android"
cd MyApp
yarn add ../RTNCenteredText
cd android
./gradlew generateCodegenArtifactsFromSchema --rerun-tasks
```

This script first adds the package to the app, in the same way iOS does. Then, after moving to the `android` folder, it invokes a Gradle task to generate the codegen.

:::note
To run the CodeGen, you need to enable the **New Architecture** in the Android app. This can be done by opening the `gradle.properties` files and by switching the `newArchEnabled` property from `false` to `true`.
:::

:::note
To run the codegen, you need to enable the **New Architecture** in the Android app. This can be done by opening the `gradle.properties` files and by switching the `newArchEnabled` property from `false` to `true`.

:::

The generated code is stored in the `MyApp/node_modules/rtn-centered-text/android/build/generated/source/codegen` folder and it has this structure:

```title="Android generated code"
codegen
├── java
│   └── com
│       └── facebook
│           └── react
│               └── viewmanagers
│                   ├── RTNCenteredTextManagerDelegate.java
│                   └── RTNCenteredTextManagerInterface.java
├── jni
│   ├── Android.mk
│   ├── CMakeLists.txt
│   ├── RTNCenteredText-generated.cpp
│   ├── RTNCenteredText.h
│   └── react
│       └── renderer
│           └── components
│               └── RTNCenteredText
│                   ├── ComponentDescriptors.h
│                   ├── EventEmitters.cpp
│                   ├── EventEmitters.h
│                   ├── Props.cpp
│                   ├── Props.h
│                   ├── ShadowNodes.cpp
│                   └── ShadowNodes.h
└── schema.json
```

You can see that the content of the `codegen/jni/react/renderer/components/RTNCenteredTextSpecs` looks similar to the files created by the iOS counterpart. Other interesting pieces are the `Android.mk` and `CMakeList.txt` files, which we need to configure the Fabric Component in the app, and the `RTNCenteredTextManagerDelegate.java` and `RTNCenteredTextManagerInterface.java` that we need to use in our manager.

See the [CodeGen](./pillars-codegen) section for further details on the generated files.

#### Write the Native Android Code

The native code for the Android side of a Fabric Components requires four pieces:

1. An `AndroidManifest.xml` file.
2. A `RTNCenteredText.java` that represents the actual view.
3. A `RTNCenteredTextManager.java` to instantiate the view.
4. A `RTNCenteredTextPackage.java` that React Native uses to configure the library.

The final structure within the Android library should be like this.

```title="Android Folder Structure"
android
├── build.gradle
└── src
    └── main
        ├── AndroidManifest.xml
        └── java
            └── com
                └── rtncenteredtext
                    ├── RTNCenteredText.java
                    ├── RTNCenteredTextManager.java
                    └── RTNCenteredTextPackage.java
```

##### AndroidManifest.xml

```xml title="AndroidManifest.xml"
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.rtncenteredtext">
</manifest>
```

This is a small manifest file that defines the package for our module.

##### RTNCenteredText.java

```java title="RTNCenteredText"
package com.rtncenteredtext;

import androidx.annotation.Nullable;
import android.content.Context;
import android.util.AttributeSet;
import android.graphics.Color;

import android.widget.TextView;
import android.view.Gravity;

public class RTNCenteredText extends TextView {

    public RTNCenteredText(Context context) {
        super(context);
        this.configureComponent();
    }

    public RTNCenteredText(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        this.configureComponent();
    }

    public RTNCenteredText(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.configureComponent();
    }

    private void configureComponent() {
        this.setBackgroundColor(Color.RED);
        this.setGravity(Gravity.CENTER_HORIZONTAL);
    }
}
```

This class represents the actual view Android is going to represent on screen. It inherit from `TextView` and we configure the basic aspects of it using a private `configureComponent()` function

##### RTNCenteredTextManager.java

```java title="RTNCenteredTextManager.java"
package com.rtncenteredtext;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReadableArray;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.ViewManagerDelegate;
import com.facebook.react.uimanager.annotations.ReactProp;
import com.facebook.react.viewmanagers.RTNCenteredTextManagerInterface;
import com.facebook.react.viewmanagers.RTNCenteredTextManagerDelegate;


@ReactModule(name = RTNCenteredTextManager.NAME)
public class RTNCenteredTextManager extends SimpleViewManager<RTNCenteredText>
        implements RTNCenteredTextManagerInterface<RTNCenteredText> {

    private final ViewManagerDelegate<RTNCenteredText> mDelegate;

    static final String NAME = "RTNCenteredText";

    public RTNCenteredTextManager(ReactApplicationContext context) {
        mDelegate = new RTNCenteredTextManagerDelegate<>(this);
    }

    @Nullable
    @Override
    protected ViewManagerDelegate<RTNCenteredText> getDelegate() {
        return mDelegate;
    }

    @NonNull
    @Override
    public String getName() {
        return RTNCenteredTextManager.NAME;
    }

    @NonNull
    @Override
    protected RTNCenteredText createViewInstance(@NonNull ThemedReactContext context) {
        return new RTNCenteredText(context);
    }

    @Override
    @ReactProp(name = "text")
    public void setText(RTNCenteredText view, @Nullable String text) {
        view.setText(text);
    }
}
```

The `RTNCenteredTextManager` is a class used by React Native to instantiate the native component. It is the class that leverage the **CodeGen** in order to implement all the proper interfaces (See the `RTNCenteredTextManagerInterface` interface in the `implements` clause) and it uses the `RTNCenteredTextManagerDelegate`.

It is also responsible to export all the constructs required by ReactNative: the class itself is annotated with `@ReactModule` and the `setText` method is annothated with `@ReactProp`.

##### RTNCenteredTextPackage.java

```java title="RTNCenteredTextPackage"
package com.rtncenteredtext;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class RTNCenteredTextPackage implements ReactPackage {

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        List<ViewManager> viewManagers = new ArrayList<>();
        viewManagers.add(new RTNCenteredTextManager(reactContext));
        return viewManagers;
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

}
```

This is the last piece of Native Code for Android. It defines the Package object that will be used by the app to load the manager.

## 4. Adding the Component To Your App

This is the last step to finally see our Fabric Component running on our app.

### Shared

To connect our Component to the app, we have to first add it as a dependency for our app. This step is required for both iOS and Android and can be done using the following command:

```sh
cd MyApp
yarn add ../RTNCenteredText
```

This command will add the `RTNCenteredText` Component to the `node_modules` of your app.

:::note
That is the same command we run in the [**CodeGen** step](#generate-the-code-ios) for iOS. If you run it previously, remember to first remove the package and then to add it back, in order to bring in the recent changes to the native code. The shell command to remove the package is:

```sh
yarn remove rn-centered-text
```

:::

### iOS - Configure Dependencies

To use the new component in iOS, we need to install the dependencies in our iOS project, given that they are changed. To do so, we need to run these commands:

```sh
cd ios
RCT_NEW_ARCH_ENABLED=1 pod install
```

This command will look for all the dependencies of the project and it will install the iOS ones. The `RCT_NEW_ARCH_ENABLED=1` instruct **Cocoapods** that it has to run some additional operations to run the **CodeGen** that is required by **Fabric**.

### Android - Configure Dependencies

Android configuration requires slightly more steps in order to be able to use our new Component.

First, we need to enable the **New Architecture**, because **Fabric** requires it to run properly. This can be done by:

1. Open the `android/gradle.properties` file
2. Scroll down to the end of the file and switch the `newArchEnabled` property from `false` to `true`.

Then, we need to instruct the `Android.mk` file that it needs to build also the new library.
This can be with these steps:

1. Open the `android/app/src/main/jni/Android.mk` file
1. Add this line to include the library at the beginning of the file:

   ```diff
   include $(REACT_ANDROID_DIR)/Android-prebuilt.mk

   # If you wish to add a custom TurboModule or Fabric component in your app you
   # will have to include the following autogenerated makefile.
   # include $(GENERATED_SRC_DIR)/codegen/jni/Android.mk

   +include $(NODE_MODULES_DIR)/rtn-centered-text/android/build/generated/source/codegen/jni/Android.mk
   include $(CLEAR_VARS)
   ```

1. In the same file, scroll down until you find a list of `libreact` libraries. There, we have to add the the library that has been generated. To do so, we need to add this line:
   ```diff
   libreact_codegen_rncore \
   +libreact_codegen_RTNCenteredText \
   libreact_debug \
   ```

:::note
The name of the library will be `libreact_codegen_<libraryname>` where `<libraryname>` is the value that has been set in the config.
Also, this step won't be necessary anymore as soon as we release a version of React Native which supports autolinking for Android.
:::

Finally, we need to configure the Fabric component registry to load the Fabric Component at runtime. This can be done by:

1. Open the `MyApp/android/app/src/main/jni/MainComponentsRegistry.cpp`
1. Add the following include:

   ```c++
   #include <react/renderer/components/RTNCenteredText/ComponentDescriptors.h>
   ```

1. Update the `sharedProviderRegistry` with this line:

   ```diff
   auto providerRegistry = CoreComponentsRegistry::sharedProviderRegistry();

   +providerRegistry->add(concreteComponentDescriptorProvider<RTNCenteredTextComponentDescriptor>());

   // Custom Fabric Components go here. You can register custom
   ```

### JS

Finally, we can read the Component in our JS application.
To do so, we have to:

1. Import the Component in the js file that uses it. So, if we want to use it in the `App.js`, we need to add this line:

   ```js title="App.js"
   import RTNCenteredText from 'rtn-centered-text/js/RTNCenteredTextNativeComponent';
   ```

2. Then, we need to use it in another React Native component. The syntax is the same as for any other component:
   ```js title="App.js"
   // ... other code
   const App: () => Node = () => {
     // ... other App code ...
     return (
       // ...other RN elements...
       <RTNCenteredText
         text="Hello World!"
         style={{ width: '100%', height: 30 }}
       />
       // ...other RN Elements
     );
   };
   ```

Now, we can run the React Native app and see our Component on the screen.
