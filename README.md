# react-native-orientation-locker-without-windows

> This is a fork of [react-native-orientation-locker](https://www.npmjs.com/package/react-native-orientation-locker) that removes the Windows implementation. This is because the Windows implementation is unnecessary, and can cause installation problems.

A react-native module that can listen on orientation changing of device, get current orientation, lock to preferred orientation. (cross-platform support)

### Feature

* lock screen orientation to PORTRAIT|LANDSCAPE-LEFT|PORTRAIT-UPSIDEDOWN|LANDSCAPE-RIGHT.
* listen on orientation changing of device
* get the current orientation of device


 

### Notice

1. RN 0.58 + Android target SDK 27 may cause 
```Issue: java.lang.IllegalStateException: Only fullscreen activities can request orientation``` problem, 
see [[#55]](https://github.com/wonday/react-native-orientation-locker/issues/55) for a solution.

2. orientationDidChange will be delayed on iPads if we set upside down to true.
Simply disable upside down for iPad and everything works like a charm ([[#78]](https://github.com/wonday/react-native-orientation-locker/issues/78) Thanks [truongluong1314520](https://github.com/truongluong1314520))

3. If you get the following build error on iOS: 
```ld: library not found for -lRCTOrientation-tvOS```
Just remove it from linked libraries and frameworks

### Installation
#### Using yarn (RN 0.60 and and above)

```
    yarn add react-native-orientation-locker-without-windows
```


#### Using yarn (RN 0.59 and and below)

```
    yarn add react-native-orientation-locker-without-windows
    react-native link react-native-orientation-locker-without-windows
```
#### Manual linking

Add following to MainApplication.java
(This will be added automatically by auto link. If not, please manually add the following )

```diff
//...
+import org.wonday.orientation.OrientationPackage;
    @Override
    protected List<ReactPackage> getPackages() {
      @SuppressWarnings("UnnecessaryLocalVariable")
      List<ReactPackage> packages = new PackageList(this).getPackages();
      // Packages that cannot be autolinked yet can be added manually here, for example:
      // packages.add(new MyReactNativePackage());
+      packages.add(new OrientationPackage());
      return packages;
    }
//...
```

#### Using CocoaPods (iOS Only)

Run ```pod install``` in the ios directory. Linking is not required in React Native 0.60 and above.

### Configuration

#### iOS

Add the following to your project's `AppDelegate.m`:

```diff
+#import "Orientation.h"

@implementation AppDelegate

// ...

+- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
+  return [Orientation getOrientation];
+}

@end
```

#### Android

Add following to android/app/src/main/AndroidManifest.xml

```diff
      <activity
        ....
+       android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
        android:windowSoftInputMode="adjustResize">

          ....

      </activity>

```

Implement onConfigurationChanged method (in `MainActivity.java`)

```diff
// ...

+import android.content.Intent;
+import android.content.res.Configuration;

public class MainActivity extends ReactActivity {

+   @Override
+   public void onConfigurationChanged(Configuration newConfig) {
+       super.onConfigurationChanged(newConfig);
+       Intent intent = new Intent("onConfigurationChanged");
+       intent.putExtra("newConfig", newConfig);
+       this.sendBroadcast(intent);
+   }

    // ......
}
```

Add following to MainApplication.java

```diff
+import org.wonday.orientation.OrientationActivityLifecycle;
  @Override
  public void onCreate() {
+    registerActivityLifecycleCallbacks(OrientationActivityLifecycle.getInstance());
  }
```

## Usage

### Imperative API

Whenever you want to use it within React Native code now you can:
`import Orientation from 'react-native-orientation-locker-without-windows';`

```js

import Orientation from 'react-native-orientation-locker-without-windows';


  _onOrientationDidChange = (orientation) => {
    if (orientation == 'LANDSCAPE-LEFT') {
      //do something with landscape left layout
    } else {
      //do something with portrait layout
    }
  };

  componentWillMount() {
    //The getOrientation method is async. It happens sometimes that
    //you need the orientation at the moment the js starts running on device.
    //getInitialOrientation returns directly because its a constant set at the
    //beginning of the js code.
    var initial = Orientation.getInitialOrientation();
    if (initial === 'PORTRAIT') {
      //do stuff
    } else {
      //do other stuff
    }
  };

  componentDidMount() {

    Orientation.getAutoRotateState((rotationLock) => this.setState({rotationLock}));
    //this allows to check if the system autolock is enabled or not.

    Orientation.lockToPortrait(); //this will lock the view to Portrait
    //Orientation.lockToLandscapeLeft(); //this will lock the view to Landscape
    //Orientation.unlockAllOrientations(); //this will unlock the view to all Orientations

    //get current UI orientation
    /*
    Orientation.getOrientation((orientation)=> {
      console.log("Current UI Orientation: ", orientation);
    });

    //get current device orientation
    Orientation.getDeviceOrientation((deviceOrientation)=> {
      console.log("Current Device Orientation: ", deviceOrientation);
    });
    */

    Orientation.addOrientationListener(this._onOrientationDidChange);
  },

  componentWillUnmount: function() {
    Orientation.removeOrientationListener(this._onOrientationDidChange);
  }
```

### Reactive component `<OrientationLocker>`

It is possible to have multiple `OrientationLocker` components mounted at the same time. The props will be merged in the order the `OrientationLocker` components were mounted. This follows the same usability of [\<StatusBar\>](https://reactnative.dev/docs/statusbar).

```js
import React, { useState } from "react";
import { Text, View } from "react-native";
import { OrientationLocker, PORTRAIT, LANDSCAPE } from "react-native-orientation-locker-without-windows";

export default function App() {
  const [showVideo, setShowVideo] = useState(true);
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <OrientationLocker
        orientation={PORTRAIT}
        onChange={orientation => console.log('onChange', orientation)}
        onDeviceChange={orientation => console.log('onDeviceChange', orientation)}
      />
      <Button title="Toggle Video" onPress={() => setShowVideo(!showVideo)} />
      {showVideo && (
        <View>
          <OrientationLocker orientation={LANDSCAPE} />
          <View style={{ width: 320, height: 180, backgroundColor: '#ccc' }}>
            <Text>Landscape video goes here</Text>
          </View>
        </View>
      )}
    </View>
  );
};
```

## Hooks
- `useOrientationChange`: hook for `addOrientationListener` event
- `useDeviceOrientationChange`: hook for `addDeviceOrientationListener` event

```
function SomeComponent() {
  useOrientationChange((o) => {
    // Handle orientation change
  });

  useDeviceOrientationChange((o) => {
    // Handle device orientation change
  });
}
```

## Events

- `addOrientationListener(function(orientation))`

When UI orientation changed, callback function will be called.
But if lockToXXX is called , callback function will be not called untill unlockAllOrientations.
It can return either `PORTRAIT` `LANDSCAPE-LEFT` `LANDSCAPE-RIGHT` `PORTRAIT-UPSIDEDOWN` `UNKNOWN`
When lockToXXX/unlockAllOrientations, it will force resend UI orientation changed event.

- `removeOrientationListener(function(orientation))`

- `addDeviceOrientationListener(function(deviceOrientation))`

When device orientation changed, callback function will be called.
When lockToXXX is called, callback function also can be called.
It can return either `PORTRAIT` `LANDSCAPE-LEFT` `LANDSCAPE-RIGHT` `PORTRAIT-UPSIDEDOWN` `UNKNOWN`

- `removeDeviceOrientationListener(function(deviceOrientation))`

- `addLockListener(function(orientation))`

When call lockToXXX/unlockAllOrientations, callback function will be called.
It can return either `PORTRAIT` `LANDSCAPE-LEFT` `LANDSCAPE-RIGHT` `UNKNOWN`
`UNKNOWN` means not be locked.

- `removeLockListener(function(orientation))`

- `removeAllListeners()`

## Functions

- `configure({ disableFaceUpDown: boolean })` (ios only)
- `lockToPortrait()`
- `lockToLandscape()`
- `lockToLandscapeLeft()`  this will lock to camera left home button right
- `lockToLandscapeRight()` this will lock to camera right home button left
- `lockToPortraitUpsideDown` only support android and Windows
- `lockToAllOrientationsButUpsideDown` only ios
- `unlockAllOrientations()`
- `getOrientation(function(orientation))`
- `getDeviceOrientation(function(deviceOrientation))`
- `getAutoRotateState(function(state))` (android only)
- `isLocked()` (lock status by this library)

orientation can return one of:

- `PORTRAIT`
- `LANDSCAPE-LEFT` camera left home button right
- `LANDSCAPE-RIGHT` camera right home button left
- `PORTRAIT-UPSIDEDOWN`
- `FACE-UP`
- `FACE-DOWN`
- `UNKNOWN`

Notice: PORTRAIT-UPSIDEDOWN is currently not supported on iOS at the moment. FACE-UP and FACE-DOWN is only supported on iOS.
