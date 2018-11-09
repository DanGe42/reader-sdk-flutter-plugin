# Getting Started with the Flutter Plugin for Reader SDK

This guide walks you through the process of setting up a new Flutter
project with Reader SDK. See the
[Flutter Reader SDK Technical Reference](reference.md)
for more detailed information about the methods available.


## Before you start

* You will need a Square account enabled for payment processing. If you have not
  enabled payment processing on your account (or you are not sure), visit
  [squareup.com/activate].
* Follow the **Install** instructions in the [Flutter Getting Started] guide to
  setup your Flutter development environment.


## Process overview

* [Step 1: Create a Flutter project](#step-1-create-a-flutter-project)
* [Step 2: Configure the Flutter plugin for Reader SDK](#step-2-configure-the-flutter-plugin-for-reader-sdk)
* [Step 3: Request Reader SDK credentials](#step-3-Request-reader-sdk-credentials)
* [Step 4: Install Reader SDK for Android](#step-4-install-reader-sdk-for-android)
* [Step 5: Install Reader SDK for iOS](#step-5-install-reader-sdk-for-ios)
* [Step 6: Implement Reader SDK authorization](#step-6-implement-reader-sdk-authorization)
* [Step 7: Implement the Checkout flow](#step-7-implement-the-checkout-flow)
* [Step 8. Implement Mobile Authorization](#step-8-implement-mobile-authorization)

Optional steps:

* [Support Contactless Readers](#support-contactless-readers)
* [Support Reader SDK deauthorization](#support-reader-sdk-deauthorization)


## Step 1: Create a Flutter project

The basic command is:

```bash
flutter create square_reader_sdk
```

See [Developing plugin packages] guide for more detailed instructions.


## Step 2: Configure the Flutter plugin for Reader SDK

1. Clone or download the `reader-sdk-flutter-plugin` repo to a folder in your
   `flutter` directory.
2. Edit your `pubspec.yaml` file to define the Reader SDK dependency:
    ```yaml
    dependencies:
        ...
        square_reader_sdk:
            path: path/to/reader/sdk/flutter/plugin
    ```


## Step 3: Request Reader SDK credentials

1. Open the [Square Application Dashboard].
1. Create a new Square application.
1. Click on the new application to bring up the Square application settings
   pages.
1. Open the **Reader SDK** page and click "Request Credentials" to generate your
   Reader SDK repository password.
1. You will need the **Application ID** and **Repository password** from the
   **Reader SDK** settings page to configure Reader SDK in the next steps.


## Step 4: Install Reader SDK for Android

To use the Flutter plugin on Android devices, you need to install Reader
SDK for Android so it is available to the Flutter library as a resource.
The key installation steps are outlined below. For more information on
installing Reader SDK for Android, see the [Reader SDK Android Setup Guide] at
[docs.connect.squareup.com].

1. Change to the Android folder (`android`) at the root of your flutter
   project.
1. Update the `gradle.properties` file in the `android` folder of your project
   to increase the max heap size provided to the Gradle daemon and set variables
   for the Square application ID and repository password:
   ```
    SQUARE_READER_SDK_APPLICATION_ID=YOUR_SQUARE_READER_APP_ID
    SQUARE_READER_SDK_REPOSITORY_PASSWORD=YOUR_SQUARE_READER_REPOSITORY_PASSWORD
    org.gradle.jvmargs=-Xmx4g
   ```
1. Reader SDK and its dependencies contain more than 65k methods, so your build
   script must enable Multidex. If your `minSdkVersion` is less than **21**, you
   also need to include the `multidex` dependency:
    ```
    android {
      defaultConfig {
        minSdkVersion 19
        targetSdkVersion 27
        multiDexEnabled true
      }
    }

    dependencies {
      // Add this dependency if your minSdkVersion < 21
      implementation 'com.android.support:multidex:1.0.3'
      // ...
    }
    ```
1. If your `minSdkVersion` is **less than 21**, you also need to include the
   `multidex` dependency:
 ```
 dependencies {
   // Add this dependency if your minSdkVersion < 21
   implementation 'com.android.support:multidex:1.0.3'
   // ...
 }
 ```
1. Configure the Multidex options:
    ```
    android {
      // ...
      dexOptions {
        // Ensures incremental builds remain fast
        preDexLibraries true
        // Required to build with Reader SDK
        jumboMode true
        // Required to build with Reader SDK
        keepRuntimeAnnotatedClasses false
      }
      // ...
    }
    ```
1. Extend the Android Application class (`android.app.Application`) and add code
   to Import and initialize Reader SDK:
    ```
    import com.squareup.sdk.reader.ReaderSdk;

    public class MainActivity extends FlutterActivity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            GeneratedPluginRegistrant.registerWith(this);
            ReaderSdk.initialize(this.getApplication());
            // Required if minSdkVersion < 21
            MultiDex.install(this.getApplicationContext());
        }
    }
    ```


## Step 5: Install Reader SDK for iOS

To use the Flutter plugin on iOS devices, you need to install Reader
SDK for iOS so it is available to the Flutter library as a resource.
The key installation steps are outlined below. For more information on
installing Reader SDK for iOS, see the [Reader SDK iOS Setup Guide] at
[docs.connect.squareup.com].

**TIP**: You can find the minimum supported Reader SDK version for iOS in the
[root README] for this repo.

1. Change to the iOS folder (`ios`) at the root of your Flutter project.
1. Download and configure the latest version of `SquareReaderSDK.framework` in
   your project root by replacing `YOUR_SQUARE_READER_APP_ID` and
   `YOUR_SQUARE_READER_REPOSITORY_PASSWORD` with your Reader SDK credentials and
   `READER_SDK_VERSION` with the Reader SDK version you are using in the code
   below. **The framework will install in the current directory**. (The
   current `READER_SDK_VERSION` is `1.0.1`.)
    ```bash
    ruby <(curl https://connect.squareup.com/readersdk-installer) install \
    --version READER_SDK_VERSION                                          \
    --app-id YOUR_SQUARE_READER_APP_ID                                    \
    --repo-password YOUR_SQUARE_READER_REPOSITORY_PASSWORD
    ```
1. Add Reader SDK to your Xcode project:
   * Open the **General** tab for your app target in Xcode.
   * Drag the newly downloaded `SquareReaderSDK.framework` into the
     **Embedded Binaries** section and click "Finish" in the modal that appears.
1. Add a Reader SDK build phase:
   1. Open the Xcode workspace or project for your application.
   1. In the **Build Phases** tab for your application target, click the **+**
      button (at the top of the pane).
   1. Select **New Run Script Phase**.
   1. Paste the following into the editor panel of the new run script:
      ```
      FRAMEWORKS="${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}"
      "${FRAMEWORKS}/SquareReaderSDK.framework/setup"
      ```
1. In Xcode, open the **General** tab for your app target and make sure the
   **Deployment Target** is set to 11.0+.
1. In Xcode, open the **General** tab for your app target and make sure the
   **Landscape Left** and **Landscape Right** device orientations are supported.
1. In Xcode, open the **Build Settings** tab for your app target and add **$(PROJECT_DIR)**
   to **Framework Search Paths**.
1. If you are on Xcode 10+, set build system to `Legacy Build System`
1. Update your Info.plist with the following key:value pairs in the **Info** tab
   for your application target to explain why your application requires these
   device permissions:
   * `NSLocationWhenInUseUsageDescription` : "This app integrates with Square
     for card processing. To protect buyers and sellers, Square requires your
     location to process payments."
   * `NSMicrophoneUsageDescription` : "This app integrates with Square for card
     processing. To swipe magnetic cards via the headphone jack, Square requires
     access to the microphone."
   * `NSBluetoothPeripheralUsageDescription` : This app integrates with Square
     for card processing. Square uses Bluetooth to connect your device to
     compatible hardware.
   * `NSCameraUsageDescription` : This app integrates with Square for card
      processing. Upload your account logo, feature photo and product images
      with the photos stored on your mobile device.
   * `NSPhotoLibraryUsageDescription` : This app integrates with Square for card
      processing. Upload your account logo, feature photo and product images
      with the photos stored on your mobile device.
1. Update the `application:didFinishLaunchingWithOptions:` method in your app
   delegate to initialize Reader SDK:
    ```
    #include "AppDelegate.h"
    #include "GeneratedPluginRegistrant.h"
    @import SquareReaderSDK;

    @implementation AppDelegate

    (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {

    // ...

    [SQRDReaderSDK initializeWithApplicationLaunchOptions:launchOptions];
    return YES;
    }

    @end
    ```

You will also need to add code to your Flutter project to request device and
microphone permissions.


## Step 6: Implement Reader SDK authorization

Add code to your Flutter project that authorizes Reader SDK:

```java
import 'package:reader_sdk.dart';

try {
  // authCode is a mobile authorization code from the Mobile Authorization API
  const Location authorizedLocation = await authorize(authCode);
} catch(ex) {
  switch(ex.code) {
    case AuthorizeErrorNoNetwork: {
      // Remind the user to connect to the network
    }
    break;
    case UsageError: {
      String errorMessage = ex.message;
      if (__DEV__) {
        errorMessage = errorMessage + '\n\nDebug Message: ${ex.debugMessage}';
        _logger.severe('${ex.code}:${ex.debugCode} ${ex.debugMessage}')
      }
      print('Error: ${errorMessage}');
    }
    break;
  }
}
```


## Step 7: Implement the Checkout flow

Add code to your Flutter project that starts the checkout flow and handles
the response. Reader SDK must be authorized before starting the checkout flow
and connecting a Reader is only required for card payments.

**Note**: You cannot start the checkout flow from a modal screen. To start
checkout, you must close the modal before calling `startCheckout`.

```java
import 'package:reader_sdk.dart';

// A checkout parameter is required for this checkout method
const CheckoutParameters checkoutParams = {
  amountMoney: {
    amount: 100,
    currencyCode: 'USD', // optional, use authorized location's currency code by default
  },
  // Optional for all following configuration
  skipReceipt: false,
  alwaysRequireSignature: true,
  allowSplitTender: false,
  note: 'ReaderSDKSample Transaction',
  tipSettings: {
    showCustomTipField: true,
    showSeparateTipScreen: false,
    tipPercentages: [15, 20, 30],
  },
  additionalPaymentTypes: ['cash', 'manual_card_entry', 'other'],
};

try {
  const CheckoutResult = await startCheckout(checkoutParams);
  // checkout finished successfully and checkoutResult is available
} catch(ex) {
  switch(ex.code) { ... }
}
```

## Step 8. Implement Mobile Authorization

In the context of Reader SDK, authorization refers to using the SDK with a
mobile authorization code from the [Mobile Authorization API]. Mobile
authorization tokens allow custom mobile apps to process payments on Square
hardware on behalf of a specific Square account for a given location.

For early development, you can also generate a mobile authorization token from
the **Reader SDK** settings page in the [Square Application Dashboard]. See the
[Mobile Authorization API] documentation for help setting up a service to
generate mobile authorization tokens for production use.


## Optional steps

### Support Contactless Readers

You do not need to write explicit code to take payment with a Magstripe Reader.

To take payments with a Contactless + Chip Reader, you must add code to your
Flutter project that starts the Reader SDK settings flow to pair the Reader.

```java
try {
  await startReaderSettings();
} catch (ex) {
  switch(ex.code) { ... }
}
```

### Support Reader SDK deauthorization

To switch Square locations or to deauthorize the current location, you must add
code to your Flutter project that deauthorizes Reader SDK.

```java
if (await canDeauthorize()) {
  try {
    await deauthorize();
    // Deauthorize finished successfully
  } catch(ex) {
    switch(ex.code) { ... }
  }
} else {
  print('Unable to deauthorize.');
}
```



[//]: # "Link anchor definitions"
[docs.connect.squareup.com]: https://docs.connect.squareup.com
[Mobile Authorization API]: https://docs.connect.squareup.com/payments/readersdk/mobile-authz-guide
[Reader SDK]: https://docs.connect.squareup.com/payments/readersdk/overview
[Square Dashboard]: https://squareup.com/dashboard/
[update policy for Reader SDK]: https://docs.connect.squareup.com/payments/readersdk/overview#readersdkupdatepolicy
[Testing Mobile Apps]: https://docs.connect.squareup.com/testing/mobile
[squareup.com/activate]: https://squareup.com/activate
[Square Application Dashboard]: https://connect.squareup.com/apps/
[Reader SDK Android Setup Guide]: https://docs.connect.squareup.com/payments/readersdk/setup-android
[Reader SDK iOS Setup Guide]: https://docs.connect.squareup.com/payments/readersdk/setup-ios
[root README]: ../README.md
[Flutter Getting Started]: https://flutter.io/docs/get-started/install
[Developing plugin packages]: https://flutter.io/docs/development/packages-and-plugins/developing-packages#edit-plugin-package
