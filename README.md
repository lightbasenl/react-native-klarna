# react-native-klarna

## Getting started

`$ yarn add react-native-klarna`
or
`$ npm install react-native-klarna --save`

### Mostly automatic installation (pre RN 0.60)

`$ react-native link react-native-klarna`

For RN >= 0.60 please follow `After either route` step for iOS and for Android within repositories block of the dependencies block add:
    ```gradle
        maven { url 'https://x.klarnacdn.net/mobile-sdk/'}
    ```

### Manual installation

#### iOS

##### CocoaPods route

1. In Podfile add `pod 'RNKlarna', :path => '../node_modules/react-native-klarna'`
2. Run `pod install`.

##### Manual route (only available for version < 0.2.1)

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-klarna` and add `RNKlarna.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNKlarna.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Drag and drop `KlarnaCheckoutSDK.framework` from `node_modules/react-native-klarna/ios/Frameworks`, (deselct copy resources) make sure that it appears in project's `Build Phases` ➜ `Link Binary With Libraries`
5. Check that Framework and Header search paths in Build Settings contain `$(SRCROOT)/../node_modules/react-native-klarna/ios/Frameworks`

##### After either route

Add the following key with your bundle name to your Info.plist:

```
    <key>ReturnURLKlarna</key>
    <string>YOUR_BUNDLE_NAME</string>
```

#### Android

1. Open `android/app/src/main/java/[...]/MainActivity.java`

- Add `import com.rnklarna.RNKlarnaPackage;` to the imports at the top of the file
- Add `new RNKlarnaPackage()` to the list returned by the `getPackages()` method

2.  Append the following lines to `android/settings.gradle`:
    ```gradle
    include ':react-native-klarna'
    project(':react-native-klarna').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-klarna/android')
    ```
3.  Insert the following lines inside the android block in `android/app/build.gradle`:
    ```gradle
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    ```
    add the following line inside the dependencies block:
    ```gradle
    implementation project(':react-native-klarna')
    ```
    and within repositories block of the dependencies block add:
    ```gradle
        maven { url 'https://x.klarnacdn.net/mobile-sdk/'}
    ```
    In summary, the following changes should be made:
    ```gradle
    android {
      ...
      compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
      }
      ...
    }
    dependencies {
      ...
      implementation project(':react-native-klarna')
      ...
      repositories {
        ...
        maven { url 'https://x.klarnacdn.net/mobile-sdk/'}
        ...
      }
    }
    ```
4.  Register an `intent-filter`:

    ```xml
    <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />

      <data android:scheme="<your-custom-scheme>" />
      <data android:host="<your-custom-host>" />
    </intent-filter>
    ```

5.  Make sure that activity is using `launchMode` `singleTask` or `singleTop`:
    ```xml
    <activity
      android:launchMode="singleTask|singleTop">
    ```

## Usage Example
Typical usage example is shown below, there is also an example app in `example/basic`

```javascript
import RNKlarna, { NativeEvent } from 'react-native-klarna';

import React, { PureComponent } from 'react';
import { connect } from 'react-redux';
...

export class KlarnaScreen extends PureComponent {
  state {
    snippet: '' // <- or initial snippet from your backend
    loadError: false,
  }

  onComplete = async (event: NativeEvent) => {
    const { signalType } = event;
    if (signalType === 'complete') {
      const { orderId } = this.props;
      /*
      1. Perform call to the backend
      2. Retrieve the order status and confirmation snippet.
      3. Update the Klarna component with the confirmation snippet once the order status is final
      4*. If an error occurs, set snippet to 'error' to dismiss loading screen
      */
     try {
      const result = await getConfirmationSnippet(orderId);
      const { newSnippet, orderStatus, loadError } = result;
      if orderStatus {
        this.setState({ snippet: newSnippet });
      } 
     } catch (error) {
       this.setState({ loadError: true });
     }
    }
  };

  render() {
    let { snippet } = this.state;
    const { loadError } = this.state;
    if (loadError) {
      snippet = 'error';
    }
    return (
      <View>
        <RNKlarna snippet={snippet} onComplete={this.onComplete} />
        ...
      </View>
    );
  }
}

```
