# react-native-pusher-push-notifications

Manage pusher interest subscriptions from within React Native JS

More information about Pusher Beams and their Swift library, `push-notifications-swift`, can be found on their [Github repo](https://github.com/pusher/push-notifications-swift).

[![npm version](https://badge.fury.io/js/react-native-pusher-push-notifications.svg)](https://badge.fury.io/js/react-native-pusher-push-notifications)

## Requirements
**This branch is only compatible with React Native >0.60.x**

## Getting started

`$ npm install react-native-pusher-push-notifications --save`

or yarn

`$ yarn add react-native-pusher-push-notifications`

### Automatic installation

React native link will install the pods required for this to work automatically.

### Manual steps required

#### iOS


** DO NOT follow the pusher.com push notification docs that detail modifying the AppDelegate.h/m files! - this package takes care of most of the steps for you**

1. Open ios/PodFile and update `platform :ios, '9.0'` to `platform :ios, '10.0'`
1.  Open `AppDelegate.m` and add:

```
    // Add this at the top of AppDelegate.m
    #import <RNPusherPushNotifications.h>

    // Add the following as a new methods to AppDelegate.m
    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
      NSLog(@"Registered for remote with token: %@", deviceToken);
      [[RNPusherPushNotifications alloc] setDeviceToken:deviceToken];
    }

    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
      [[RNPusherPushNotifications alloc] handleNotification:userInfo];
    }

    -(void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
      NSLog(@"Remote notification support is unavailable due to error: %@", error.localizedDescription);
    }
```

##### Possible Issues

- `linker command failed with exit code 1 (use -v to see invocation)`
    1. Open ios/YourAppName.xcodeproj in XCode
    2. Right click on Your App Name in the Project Navigator on the left, and click "New File…"
    3. Create a single empty Swift file to the project (make sure that Your App Name target is selected when adding)
    4. when Xcode asks, press Create Bridging Header and do not remove Swift file then. re-run your build.


#### Android

Refer to https://docs.pusher.com/beams/reference/android for up-to-date Pusher Beams installation 
instructions (summarized below):

1. Update `android/build.gradle`

    ```gradle
    buildscript {
        // ...
        dependencies {
            // ...
            // Add this line
            classpath('com.google.gms:google-services:4.3.0')
        }
    }    
    ```

2. Add this to `android/app/build.gradle`:

    ```gradle
    
    // add to plugins
    plugins {
        ...
        id('com.google.gms.google-services')
    }
    
    dependencies {
        ...
        implementation 'com.google.firebase:firebase-messaging:20.0.0'
        implementation 'com.pusher:push-notifications-android:1.4.4'
    }
    
    ```

3. Set up `android/app/google-services.json`

    This file is generated via [Google Firebase](https://console.firebase.google.com) console when creating a new app. Setup your app there and download the file.
    
    Pusher Beams requires a FCM secret, this is also found under Cloud Messaging in Google Firebase.

4. Add react-native.config.js to root of react-native directory
    ```
    module.exports = {
        dependencies: {
            "react-native-pusher-push-notifications": {
                platforms: {
                    android: null // this skips autolink for android
                }
            }
        }
    };
    
    ```


## Implementation

In typescript:

```typescript jsx
import {Platform} from "react-native";
import {PUSHER_BEAMS_INSTANCE_ID} from "react-native-dotenv";
import RNPusherPushNotifications from "react-native-pusher-push-notifications";

const init= (): void => {
    RNPusherPushNotifications.setInstanceId(PUSHER_BEAMS_INSTANCE_ID);

    RNPusherPushNotifications.on("notification", handleNotification);
    RNPusherPushNotifications.setOnSubscriptionsChangedListener(onSubscriptionsChanged);
};

const subscribe = (interest: string): void => {
    console.log(`Subscribing to "${interest}"`);
    RNPusherPushNotifications.subscribe(
        interest,
        (statusCode, response) => {
            console.error(statusCode, response);
        },
        () => {
            console.log(`CALLBACK: Subscribed to ${interest}`);
        }
    );
};

const handleNotification = (notification: any): void => {
    console.log(notification);
    if (Platform.OS === "ios") {
        console.log("CALLBACK: handleNotification (ios)");
    } else {
        console.log("CALLBACK: handleNotification (android)");
        console.log(notification);
    }
};

const onSubscriptionsChanged = (interests: string[]): void => {
    console.log("CALLBACK: onSubscriptionsChanged");
    console.log(interests);
}
```

## Usage

```ecmascript 6
// Import module
import RNPusherPushNotifications from 'react-native-pusher-push-notifications';

// Get your interest
const donutsInterest = 'debug-donuts';

// Initialize notifications
export const init = () => {
  // Set your app key and register for push
  RNPusherPushNotifications.setInstanceId(CONSTANTS.PUSHER_INSTANCE_ID);

  // Init interests after registration
  RNPusherPushNotifications.on('registered', () => {
    subscribe(donutsInterest);
  });

  // Setup notification listeners
  RNPusherPushNotifications.on('notification', handleNotification);
};

// Handle notifications received
const handleNotification = notification => {
  console.log(notification);

  // iOS app specific handling
  if (Platform.OS === 'ios') {
    switch (notification.appState) {
      case 'inactive':
      // inactive: App came in foreground by clicking on notification.
      //           Use notification.userInfo for redirecting to specific view controller
      case 'background':
      // background: App is in background and notification is received.
      //             You can fetch required data here don't do anything with UI
      case 'active':
      // App is foreground and notification is received. Show a alert or something.
      default:
        break;
    } else {
        // console.log("android handled notification...");
    }
  }
};

// Subscribe to an interest
const subscribe = interest => {
  // Note that only Android devices will respond to success/error callbacks
  RNPusherPushNotifications.subscribe(
    interest,
    (statusCode, response) => {
      console.error(statusCode, response);
    },
    () => {
      console.log('Success');
    }
  );
};

// Unsubscribe from an interest
const unsubscribe = interest => {
  RNPusherPushNotifications.unsubscribe(
    interest,
    (statusCode, response) => {
      console.tron.logImportant(statusCode, response);
    },
    () => {
      console.tron.logImportant('Success');
    }
  );
};
```

## iOS only methods

```ecmascript 6
// Set interests
const donutInterests = ['debug-donuts', 'debug-general'];
const setSubscriptions = donutInterests => {
  // Note that only Android devices will respond to success/error callbacks
  RNPusherPushNotifications.setSubscriptions(
    donutInterests,
    (statusCode, response) => {
      console.error(statusCode, response);
    },
    () => {
      console.log('Success');
    }
  );
};
```

# Sample Payload

`POST` to `https://{pusher_instance_id}.pushnotifications.pusher.com/publish_api/v1/instances/{pusher_instance_id}/publishes` with headers:
```headers
    Content-Type: application/json
    Authorization: Bearer {pusher_secret_key}
```
```json
{
  "interests": [
  	"debug-donuts"
  ],
  "apns": {
    "aps": {
    	"alert" : {
	      "title": "iOS Notification",
	      "body": "Hello ios user",
    	},
    "badge": 12
    }
  },
  "fcm": {
    "notification": {
      "title": "Android notification",
      "body": "Hello android user"
    }
  }
}

```

## Increment Badge number

The APS data sent to Pusher Beams and then to Apple have an option `badge`. This will update your apps badge counter to the current number. If you send 1, the badge will show 1. This means you need to handle notification read status in your backend and when pushing update to the current number.

By adding `incrementBadge` you can increment the badge number without having to deal with your backend.

```
{
  aps: {
    data: {
      incrementBadge: true
    }
  }
}
```
