[![Circle CI](https://circleci.com/gh/amplitude/Amplitude-iOS/tree/master.svg?style=badge&circle-token=e1b2a7d2cd6dd64ac3643bc8cb2117c0ed5cbb75)](https://circleci.com/gh/amplitude/Amplitude-iOS/tree/master)

Amplitude iOS SDK
====================

An iOS SDK for tracking events and revenue to [Amplitude](http://www.amplitude.com).

A [demo application](https://github.com/amplitude/iOS-Demo) is available to show a simple integration.

# Setup #
1. If you haven't already, go to https://amplitude.com and register for an account. You will receive an API Key.

2. [Download the source code](https://github.com/amplitude/Amplitude-iOS/archive/master.zip) and extract the zip file. Alternatively, you can pull directly from GitHub. If you use CocoaPods, add the following line to your Podfile: `pod 'Amplitude-iOS', '~> 3.5.0'`. If you are using CocoaPods, you may skip steps 3 and 4.

3. Copy the `Amplitude` sub-folder into the source of your project in Xcode. Check "Copy items into destination group's folder (if needed)".

4. Amplitude's iOS SDK requires the SQLite library, which is included in iOS but requires an additional build flag to enable. In your project's `Build Settings` and your Target's `Build Settings`, under `Linking` -> `Other Linker Flags`, add the flag `-lsqlite3`.

5. In every file that uses analytics, import Amplitude.h at the top:
    ``` objective-c
    #import "Amplitude.h"
    ```

6. In the application:didFinishLaunchingWithOptions: method of your YourAppNameAppDelegate.m file, initialize the SDK:
    ``` objective-c
    [[Amplitude instance] initializeApiKey:@"YOUR_API_KEY_HERE"];
    ```

7. To track an event anywhere in the app, call:
    ``` objective-c
    [[Amplitude instance] logEvent:@"EVENT_IDENTIFIER_HERE"];
    ```

8. Events are saved locally. Uploads are batched to occur every 30 events and every 30 seconds, as well as on app close. After calling logEvent in your app, you will immediately see data appear on the Amplitude Website.

# Tracking Events #

It's important to think about what types of events you care about as a developer. You should aim to track between 20 and 200 types of events within your app. Common event types are different screens within the app, actions the user initiates (such as pressing a button), and events you want the user to complete (such as filling out a form, completing a level, or making a payment). Contact us if you want assistance determining what would be best for you to track.

# Tracking Sessions #

A session is a period of time that a user has the app in the foreground. Sessions within 5 minutes of each other are merged into a single session. In the iOS SDK, sessions are tracked automatically. When the SDK is initialized, it determines whether the app is launched into the foreground or background and starts a new session if launched in the foreground. A new session is created when the app comes back into the foreground after being out of the foreground for 5 minutes or more. If the app is in the background and an event is logged, then a new session is created if more than 5 minutes has passed since the app entered the background or when the last event was logged (whichever occured last). Otherwise the background event logged will be part of the current session.

You can adjust the time window for which sessions are extended by changing the variable minTimeBetweenSessionsMillis:
``` objective-c
[Amplitude instance].minTimeBetweenSessionsMillis = 30 * 60 * 1000; // 30 minutes
[[Amplitude instance] initializeApiKey:@"YOUR_API_KEY_HERE"];
```

By default start and end session events are no longer sent. To renable add this line before initializing the SDK:
``` objective-c
[[Amplitude instance] setTrackingSessionEvents:YES];
[[Amplitude instance] initializeApiKey:@"YOUR_API_KEY_HERE"];
```

You can also log events as out of session. Out of session events have a session_id of -1 and are not considered part of the current session, meaning they do not extend the current session. This might be useful for example if you are logging events triggered by push notifications. You can log events as out of session by setting input parameter outOfSession to true when calling logEvent.

``` objective-c
[[Amplitude instance] logEvent:@"EVENT_IDENTIFIER_HERE" withEventProperties:nil outOfSession:true];
```

# Setting Custom User IDs #

If your app has its own login system that you want to track users with, you can call `setUserId:` at any time:

``` objective-c
[[Amplitude instance] setUserId:@"USER_ID_HERE"];
```

A user's data will be merged on the backend so that any events up to that point on the same device will be tracked under the same user. Note: if a user logs out, or you want to log events under an anonymous user, you can also clear the user ID by calling `setUserId` with input `nil`:

``` objective-c
[[Amplitude instance] setUserId:nil]; // not string @"nil"
```

You can also add the user ID as an argument to the `initializeApiKey:` call:

``` objective-c
[[Amplitude instance] initializeApiKey:@"YOUR_API_KEY_HERE" userId:@"USER_ID_HERE"];
```

# Setting Event Properties #

You can attach additional data to any event by passing a NSDictionary object as the second argument to logEvent:withEventProperties:

``` objective-c
NSMutableDictionary *eventProperties = [NSMutableDictionary dictionary];
[eventProperties setValue:@"VALUE_GOES_HERE" forKey:@"KEY_GOES_HERE"];
[[Amplitude instance] logEvent:@"Compute Hash" withEventProperties:eventProperties];
```

Note: the keys should be of type NSString, and the values should be of type NSString, NSNumber, NSArray, NSDictionary, or NSNull. You will see a warning if you try to use an unsupported type.

# User Properties and User Property Operations #

The SDK supports the operations set, setOnce, unset, and add on individual user properties. The operations are declared via a provided `AMPIdentify` interface. Multiple operations can be chained together in a single `AMPIdentify` object. The `AMPIdentify` object is then passed to the Amplitude client to send to the server. The results of the operations will be visible immediately in the dashboard, and take effect for events logged after. Note, each
operation on the `AMPIdentify` object returns the same instance, allowing you to chain multiple operations together.

To use the `AMPIdentify` interface, you will first need to include the header:
``` objective-c
#import "AMPIdentify.h"
```

1. `set`: this sets the value of a user property.

    ``` objective-c
    AMPIdentify *identify = [[[AMPIdentify identify] set:@"gender" value:@"female"] set:@"age" value:[NSNumber numberForInt:20]];
    [[Amplitude instance] identify:identify];
    ```

2. `setOnce`: this sets the value of a user property only once. Subsequent `setOnce` operations on that user property will be ignored. In the following example, `sign_up_date` will be set once to `08/24/2015`, and the following setOnce to `09/14/2015` will be ignored:

    ``` objective-c
    AMPIdentify *identify1 = [[AMPIdentify identify] setOnce:@"sign_up_date" value:@"09/06/2015"];
    [[Amplitude instance] identify:identify1];

    AMPIdentify *identify2 = [[AMPIdentify identify] setOnce:@"sign_up_date" value:@"10/06/2015"];
    [[Amplitude instance] identify:identify2];
    ```

3. `unset`: this will unset and remove a user property.

    ``` objective-c
    AMPIdentify *identify = [[[AMPIdentify identify] unset:@"gender"] unset:@"age"];
    [[Amplitude instance] identify:identify];
    ```

4. `add`: this will increment a user property by some numerical value. If the user property does not have a value set yet, it will be initialized to 0 before being incremented.

    ``` objective-c
    AMPIdentify *identify = [[[AMPIdentify identify] add:@"karma" value:[NSNumber numberWithFloat:0.123]] add:@"friends" value:[NSNumber numberWithInt:1]];
    [[Amplitude instance] identify:identify];
    ```

5. `append`: this will append a value or values to a user property. If the user property does not have a value set yet, it will be initialized to an empty list before the new values are appended. If the user property has an existing value and it is not a list, it will be converted into a list with the new value appended.

    ``` objective-c
    NSMutableArray *array = [NSMutableArray array];
    [array addObject:@"some_string"];
    [array addObject:[NSNumber numberWithInt:56]];
    AMPIdentify *identify = [[[AMPIdentify identify] append:@"ab-tests" value:@"new-user-test"] append:@"some_list" value:array];
    [[Amplitude instance] identify:identify];
    ```

Note: if a user property is used in multiple operations on the same `Identify` object, only the first operation will be saved, and the rest will be ignored. In this example, only the set operation will be saved, and the add and unset will be ignored:

``` objective-c
AMPIdentify *identify = [[[[AMPIdentify identify] set:@"karma" value:[NSNumber numberWithInt:10]] add:@"friends" value:[NSNumber numberWithInt:1]] unset:@"karma"];
    [[Amplitude instance] identify:identify];
```

### Arrays in User Properties ###

The SDK supports arrays in user properties. Any of the user property operations above (with the exception of `add`) can accept an NSArray or an NSMutableArray. You can directly `set` arrays, or use `append` to generate an array.

``` objective-c
NSMutableArray *colors = [NSMutableArray array];
[colors addObject:@"rose"];
[colors addObject:@"gold"];
NSMutableArray *numbers = [NSMutableArray array];
[numbers addObject:[NSNumber numberWithInt:4]];
[numbers addObject:[NSNumber numberWithInt:5]];
AMPIdentify *identify = [[[[AMPIdentify identify] set:@"colors" value:colors] append:@"ab-tests" value:@"campaign_a"] append:@"existing_list" value:numbers];
[[Amplitude instance] identify:identify];
```

### Setting Multiple Properties with `setUserProperties` ###

You may use `setUserProperties` shorthand to set multiple user properties at once. This method is simply a wrapper around `AMPIdentify set` and `identify`.

``` objective-c
NSMutableDictionary *userProperties = [NSMutableDictionary dictionary];
[userProperties setValue:@"VALUE_GOES_HERE" forKey:@"KEY_GOES_HERE"];
[userProperties setValue:@"OTHER_VALUE_GOES_HERE" forKey:@"OTHER_KEY_GOES_HERE"];
[[Amplitude instance] setUserProperties:userProperties];
```

### Clearing User Properties with `clearUserProperties` ###

You may use `clearUserProperties` to clear all user properties at once. Note: the result is irreversible!

``` objective-c
[[Amplitude instance] clearUserProperties];
```

# Allowing Users to Opt Out

To stop all event and session logging for a user, call setOptOut:

``` objective-c
[[Amplitude instance] setOptOut:YES];
```

Logging can be restarted by calling setOptOut again with enabled set to NO.
No events will be logged during any period opt out is enabled, even after opt
out is disabled.

# Tracking Revenue #

To track revenue from a user, call

``` objective-c
[[Amplitude instance] logRevenue:@"productIdentifier" quantity:1 price:[NSNumber numberWithDouble:3.99]]
```

after a successful purchase transaction. `logRevenue:` takes a string to identify the product (can be pulled from `SKPaymentTransaction.payment.productIdentifier`). `quantity:` takes an integer with the quantity of product purchased. `price:` takes a NSNumber with the dollar amount of the sale as the only argument. This allows us to automatically display data relevant to revenue on the Amplitude website, including average revenue per daily active user (ARPDAU), 7, 30, and 90 day revenue, lifetime value (LTV) estimates, and revenue by advertising campaign cohort and daily/weekly/monthly cohorts.

**To enable revenue verification, copy your iTunes Connect In App Purchase Shared Secret into the manage section of your app on Amplitude. You must put a key for every single app in Amplitude where you want revenue verification.**

Then call

``` objective-c
[[Amplitude instance] logRevenue:@"productIdentifier" quantity:1 price:[NSNumber numberWithDouble:3.99 receipt:receiptData]
```

after a successful purchase transaction. `receipt:` takes the receipt NSData from the app store. For details on how to obtain the receipt data, see [Apple's guide on Receipt Validation](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1).

# Tracking Events to Multiple Amplitude Apps #

The Amplitude iOS SDK supports logging events to multiple Amplitude apps (multiple API keys). If you want to log events to multiple Amplitude apps, you need to use separate instances for each Amplitude app. Each new instance created will have its own apiKey, userId, deviceId, and settings.

You will need to assign a name to each Amplitude app / instance, and use that name consistently when fetching that instance to call functions. **IMPORTANT: Once you have chosen a name for that instance you cannot change it.** Every instance's data and settings are tied to its name, and you will need to continue using that instance name for all future versions of your app to maintain data continuity, so chose your instance names carefully. Note these names do not need to correspond to the names of your apps in the Amplitude dashboards, but they need to remain consistent throughout your code. You also need to be sure that each instance is initialized with the correct apiKey.

Instance names must be nonnil and nonempty strings. The names are case-insensitive. You can fetch each instance by name by calling `[Amplitude instanceWithName:@"INSTANCE_NAME"]`.

As mentioned before, each new instance created will have its own apiKey, userId, deviceId, and settings. **You will have to reconfigure all the settings for each instance.** For example if you want to track session events you would have to call `setTrackingSessionEvents:YES` on each instance. This does give you the freedom to have different settings for each instance.

### Backwards Compatibility - Upgrading from a Single Amplitude App to Multiple Apps ###

If you were tracking users with a single app before v3.6.0, you might be wondering what will happen to existing data, existing settings, and returning users (users who already have a deviceId and/or userId). All of the historical data and settings are maintained on the `default` instance, which is fetched without an instance name: `[Amplitude instance]`. This is the way you are used to interacting with the Amplitude SDK, which means all of your existing tracking code should work as before.

### Example of how to Set Up and Log Events to Two Separate Apps ###

``` objective-c
[[Amplitude instance] initializeApiKey:@"12345"]; // existing app, existing settings, and existing API key
[[Amplitude instanceWithName:@"new_app"] initializeApiKey:@"67890"]; // new app, new API key

[[Amplitude instanceWithName:@"new_app"] setUserId:@"joe@gmail.com"]; // need to reconfigure new app
[[Amplitude instanceWithName:@"new_app"] logEvent:@"Clicked"];

AMPIdentify *identify = [[AMPIdentify identify] add:@"karma" value:[NSNumber numberWithInt:1]];
[[Amplitude instance] identify:identify];
[[Amplitude instance] logEvent:@"Viewed Home Page"];
```

### Synchronizing Device Ids Between Apps ###

As mentioned before, each new instance will have its own deviceId. If you want your apps to share the same deviceId, you can do so *after initialization* via the `getDeviceId` and `setDeviceId` methods. Here's an example of how to copy the existing deviceId to the `new_app` instance:
``` objective-c
NSString *deviceId = [[Amplitude instance] getDeviceId]; // existing deviceId
[[Amplitude instanceWithName:@"new_app"] setDeviceId:deviceId]; // transferring existing deviceId to new app
```

# Swift #

This SDK will work with Swift. If you are copying the source files or using CocoaPods without the `use_frameworks!` directive, you should create a bridging header as documented [here](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html) and add the following line to your bridging header:

``` objective-c
#import "Amplitude.h"
```

If you have `use_frameworks!` set, you should not use a bridging header and instead use the following line in your swift files:

``` swift
import Amplitude_iOS
```

In either case, you can call Amplitude methods with `Amplitude.instance().method(...)`

# Advanced #
This SDK automatically grabs useful data from the phone, including app version, phone model, operating system version, and carrier information.

### Location Tracking ###
If the user has granted your app location permissions, the SDK will also grab the location of the user. Amplitude will never prompt the user for location permissions itself, this must be done by your app.

Amplitude only polls for a location once on startup of the app, once on each app open, and once when the permission is first granted. There is no continuous tracking of location, although you can force Amplitude to grab the latest location by calling `[[Amplitude instance] updateLocation]`. Note this does consume more resources on the user's device, so use this wisely.

If you wish to disable location tracking done by the app, you can call `[[Amplitude instance] disableLocationListening]` at any point. If you want location tracking disabled on startup of the app, call disableLocationListening before you call `initializeApiKey:`. You can always reenable location tracking through Amplitude with `[[Amplitude instance] enableLocationListening]`.

### Custom Device IDs ###
Device IDs are randomly generated. You can, however, choose to instead use the identifierForVendor (if available) by calling `[[Amplitude instance] useAdvertisingIdForDeviceId]` before initializing with your API key. You can also retrieve the Device ID that Amplitude uses with `[[Amplitude instance] getDeviceId]`.

If you have your own system for tracking device IDs and would like to set a custom device ID, you can do so with `[[Amplitude instance] setDeviceId:@"CUSTOM_DEVICE_ID"];` **Note: this is not recommended unless you really know what you are doing.** Make sure the device ID you set is sufficiently unique (we recommend something like a UUID - see `[AMPUtils generateUUID]` for an example on how to generate) to prevent conflicts with other devices in our system.

### ARC ###
This code will work with both ARC and non-ARC projects. Preprocessor macros are used to determine which version of the compiler is being used.

### SSL pinning ###
The SDK includes support for SSL pinning, but it is undocumented and recommended against unless you have a specific need. Please contact Amplitude support before you ship any products with SSL pinning enabled so that we are aware and can provide documentation and implementation help.
