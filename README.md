# Reat Native Push Notification template

![Alt text](./qhwb.gif)
 
## How to install
1. Install npm package
```
yarn install 
```

2. Following [this](https://firebase.google.com/docs/android/setup) tutorial make the following:
- add generated `google-services.json` file to `android/app` folder

## API

### getDeviceToken()

Returns device FCM token.

Example:
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const token = await PushNotificationAndroid.getDeviceToken();
console.log(token); // 7rilPUr_OJBvggou...
```

### cancelScheduledNotification(notificationId)

Cancels scheduled notification by notification id. Scheduled notification can be created if you pass `showAt` param to `show()` method.

- **notificationId**: notification id which should be cancelled

Example:
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const notificationId = 1;
PushNotificationAndroid.cancelScheduledNotification(notificationId);
```

### createChannel(channelId, channelName, channelDesc, channelImportance)

Creates a notification channel. For android >= 8 (SDK >= 26) channels are required when you show a push notification. NOTICE: you should call this method on app init and pass *channel id* from your manifest file.

- **channelId**: channel id. Used on notification show.
- **channelName**: human readable channel name. Channel name is displayed in app notification settings.
- **channelDesc**: channel description.
- **channelImportance**: channel importance, the more importance the more chances that user will see a notification. Available values:
	- PushNotificationAndroid.IMPORTANCE_NONE
	- PushNotificationAndroid.IMPORTANCE_MIN
	- PushNotificationAndroid.IMPORTANCE_LOW
	- PushNotificationAndroid.IMPORTANCE_DEFAULT
	- PushNotificationAndroid.IMPORTANCE_HIGH
	- PushNotificationAndroid.IMPORTANCE_MAX

Example:
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const channelId = "my_channel_id";
const channelName = "my_channel_name";
const channelDesc = "my_channel_desc";
const channelImportance = PushNotificationAndroid.CHANNEL_IMPORTANCE_DEFAULT;
PushNotificationAndroid.createChannel(channelId, channelName, channelDesc, channelImportance);
```

### show(notificationId, template, channelId, data, priority = PushNotificationAndroid.PRIORITY_DEFAULT, badgeNumber = 0, showAt = 0)

Shows a push notification. You can use this method locally. This method is also called when remote notification is received.

- **notificationId**: notification id. We need this id in case we would want to modify a notification.
- **template**: template id. There are 2 templates available:
	- PushNotificationAndroid.TEMPLATE_COMMON: standard push notification with title and message, 
	- PushNotificationAndroid.TEMPLATE_EVENT: push notification with custom template. Consists of: button with checkmark, small media image, url that should be opened on notification content click, title and text. When your app receive a remote notification with `media` or `url` data attributes then PushNotificationAndroid.TEMPLATE_EVENT will be used.
- **channelId**: channel id. By default you should use the one from android manifest (as remote notifications use the same one).
- **data**: notification data attributes.
- **priority**: notification priority. Android >= 8 (SDK >= 26) uses notification channels to set priority. Android < 8 (SDK < 26) sets priority directly on a notification. So this priority field is for compatibility with Android < 8 (SDK < 26). Available notification priorities:
	- PushNotificationAndroid.PRIORITY_MIN
	- PushNotificationAndroid.PRIORITY_LOW
	- PushNotificationAndroid.PRIORITY_DEFAULT
	- PushNotificationAndroid.PRIORITY_HIGH
	- PushNotificationAndroid.PRIORITY_MAX
- **badgeNumber**: badge number on the app icon
- **showAt**: unix timestamp when push notification should be shown (can be used for scheduled notifications). If this param is set to 0 then notification is shown immediately. NOTICE: scheduled notifications work only for android >= 6 (SDK >= 23).

Example (default notification with title and message):
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const notificationId = 1;
const template = PushNotificationAndroid.TEMPLATE_COMMON;
const channelId = 'my_channel_id';
const data = {
	title: "my title",
	message: "my message"
};
const priority = PushNotificationAndroid.PRIORITY_DEFAULT;
const badgeNumber = 10;
const showAt = 0;
PushNotificationAndroid.show(notificationId, template, channelId, data, priority, badgeNumber, showAt);
```

Example (custom notification template with checkmark button, media image, url that will be opened on notification content click, title and message):
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const template = PushNotificationAndroid.TEMPLATE_EVENT;
const channelId = 'my_channel_id';
const data = {
	title: "title", // optional (can be null)
	message: "my message", // optional (can be null)
	media: "http://red-msk.ru/wp-content/uploads/2019/02/canva-photo-editor-22.png", // optional (can be null)
	url: "https://google.com" // optional (can be null)
};
const priority = PushNotificationAndroid.PRIORITY_DEFAULT;
const badgeNumber = 10;
const showAt = 0;
PushNotificationAndroid.show(notificationId, template, channelId, data, priority, badgeNumber, showAt);
```

Example (scheduled notification which should be shown after 60 seconds have passed):
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
import moment from 'moment';
const notificationId = 1;
const template = PushNotificationAndroid.TEMPLATE_COMMON;
const channelId = 'my_channel_id';
const data = {
	title: "my title",
	message: "my message"
};
const priority = PushNotificationAndroid.PRIORITY_DEFAULT;
const badgeNumber = 10;
const showAt = moment().utc().unix() + 60;
PushNotificationAndroid.show(notificationId, template, channelId, data, priority, badgeNumber, showAt);
```

## How to add a custom template to this repository

All new notification templates are added in native code. Let's add a weather notification template which shows current temperature (with a single text view).

1. Create a new notification template file `notification_template_weather.xml` inside `android/src/main/res/layout` folder with the following content:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="64dp">

    <TextView
        android:id="@+id/textViewTemperature"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1" />
</LinearLayout>
```

We show a single text view with id `textViewTemperature`.

NOTICE: by default android notification height is 64dp so you should follow this rule.

NOTICE: not all layouts and widgets are supported in a custom push notification template. More info [here](https://developer.android.com/reference/android/widget/RemoteViews.html).

2. Modify the `Templates` object in `PushNotificationModule.kt` file:
```
object Templates {
    val COMMON = 0
    val EVENT = 1
    val WEATHER = 2 // your new template
}
```

3. Modify the `getConstants()` method in `PushNotificationModule.kt` file so that your template could be accessible from React Native:
```
override fun getConstants(): MutableMap<String, Any> {
    val constants = mutableMapOf<String, Any>()
    // ... other constants
    constants.put("TEMPLATE_WEATHER", Templates.WEATHER) // your new template
    return constants
}
```

4. Assign notification data attributes to the xml temlpate in `show()` method in `PushNotificationModule.`:
```
@ReactMethod
fun show(notificationId: Int, template: Int, channelId: String, data: ReadableMap, priority: Int = NotificationCompat.PRIORITY_DEFAULT, badgeNumber: Int = 0, showAt: Int = 0) {
    // ... other templates
    if(template == Templates.WEATHER) {
        val remoteViews = RemoteViews(reactApplicationContext.packageName, R.layout.notification_template_weather)
        // assign "temperature" notification data attribute to "textViewTemperature" TextView in xml template
        remoteViews.setTextViewText(R.id.textViewTemperature, data.getString("temperature"))
        builder.setContent(remoteViews)
    }
    // ... show notification code
}
```

5. Modify `CustomFirebaseMessagingService.kt` service so that remote notifications could use your template. Add the following code to the `onMessageReceived()` method:
```
override fun onMessageReceived(remoteMessage: RemoteMessage) {
    // ... other code
    // if notification has "temperature" data attribute then use WEATHER template
    if(!remoteMessage.data.get("temperature").isNullOrEmpty()) {
        templateId = PushNotificationModule.Templates.WEATHER
    }
    module.show(notificationId, templateId, channelId, rnMap)
}
```

6. Now all remote notifications with "temperature" data attribute by default will use WEATHER notification template. You can show local notification with the weather template using the following code:
```
import { PushNotificationAndroid } from '@cryptoticket/react-native-push-notification';
const notificationId = 1;
const template = PushNotificationAndroid.TEMPLATE_WEATHER;
const channelId = 'my_channel_id';
const data = {
    temperature: "+24 degrees"
};
const priority = PushNotificationAndroid.PRIORITY_DEFAULT;
const badgeNumber = 10;
const showAt = 0;
PushNotificationAndroid.show(notificationId, template, channelId, data, priority, badgeNumber, showAt);
```

## How to add deep links support
By default main app screen is opened when user presses on any notification. With this package you can add the ability to open app via deep link on notification press. For example, if you receive `content_id` in notification data attributes then on notification press you can open app by deep link (ex: `app://content/1`) and inside your app handle this link as you need.

1. Add intent filter to `MainActivity` section to `AndroidManifest.xml` file:
```
<activity
  android:name=".MainActivity"
  android:label="@string/app_name"
  android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
  android:windowSoftInputMode="adjustResize">
    <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <data android:scheme="app" android:host="content" />
    </intent-filter>
</activity>
```

Now your app should be able to handle deep links, for example `app://content/1`. You can open app from deep link via adb console command `adb shell am start -W -a android.intent.action.VIEW -d "app://content/1" com.example`(use your app package name).

2. Create a new native class that should extend existing broadcast receiver and override `onNotificationPress()` function:
```
package com.example;

import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;

import com.cryptoticket.reactnativepushnotification.PushNotificationBroadcastReceiver;

public class CustomPushNotificationBroadcastReceiver extends PushNotificationBroadcastReceiver {
    @Override
    public void onNotificationPress(Context context, Intent intent) {
        boolean shouldHandleDeepLink = true;
        if(shouldHandleDeepLink) {
            // list all notification data attributes
            Bundle bundle = intent.getExtras();
            if (bundle != null) {
                for (String key : bundle.keySet()) {
                    Log.d("CUSTOM RECEIVER", key + " : " + (bundle.get(key) != null ? bundle.get(key) : "NULL"));
                }
            }
            // open app by deep link
            Intent mainIntent = new Intent(Intent.ACTION_VIEW);
            mainIntent.setData(Uri.parse("app://content/1"));
            mainIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(mainIntent);
        } else {
            // open main app activity by default
            super.onNotificationPress(context, intent);
        }
    }
}

```

So you may check notification data attributes and depening on them open app from deep link or use the default behavior which opens app's main screen.

3. Tell the package the class name of your broadcast receiver. Add the following xml to `AndroidManifest.xml`:
```
<!-- start optional notification settings -->
  <meta-data
  	android:name="com.cryptoticket.reactnativepushnotification.default_broadcast_receiver"
  	android:value="com.example.CustomPushNotificationBroadcastReceiver" />
<!-- end optional notification settings -->
```

4. Tell the package that you're going to use your own broadcast receiver. Update the `receiver` name in `AndroidManifest.xml`.

Old:
```
<receiver android:name="com.cryptoticket.reactnativepushnotification.PushNotificationBroadcastReceiver" android:exported="true">
```

New:
```
<receiver android:name=".CustomPushNotificationBroadcastReceiver" android:exported="true">
```

5. Now you can get deep link in your app via [React Native Linking library](https://reactnative.dev/docs/linking).


## Troubleshooting
- **java.lang.IllegalStateException: GeneratedAppGlideModuleImpl is implemented incorrectly. If you've manually implemented this class, remove your implementation. The Annotation processor will generate a correct implementation.**

You may get this error on app build if one of your npm dependencies uses glide. Solution is [here](https://github.com/DylanVann/react-native-fast-image/blob/master/docs/app-glide-module.md). You should add the following code to your `android/build.gradle`:
```
project.ext {
    excludeAppGlideModule = true
}
```
- **twilio remote notifications**

By default twilio wraps notification body(message) in `twi_body` param. This package expects param `message` to be a notification body, not `twi_body`. Solution: copy `twi_body` notification attribute to `message` attribute on remote notification receive.

Create a new class and extend existing `CustomFirebaseMessagingService`:
```
package com.example;

import com.cryptoticket.reactnativepushnotification.CustomFirebaseMessagingService;
import com.google.firebase.messaging.RemoteMessage;
import java.util.Map;

/**
 * Extends react-native-push-notification message service.
 */
public class MainMessagingService extends CustomFirebaseMessagingService {
    
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        /**
         * By default twilio sends notification body message in "twi_body" param.
         * Our lib needs notification message param to be called "message",
         * so we copy "twi_body" to "message" in remote message data map.
         */
        if(remoteMessage.getData().containsKey("twi_body")) {
            remoteMessage.getData().put("message", remoteMessage.getData().get("twi_body"));
        }
        // call parent method
        super.onMessageReceived(remoteMessage);
    }
}

```
Delete the default firebase service from the manifest:
```
<service
   android:name="com.cryptoticket.reactnativepushnotification.CustomFirebaseMessagingService"
   android:exported="false">
     <intent-filter>
     	<action android:name="com.google.firebase.MESSAGING_EVENT" />
     </intent-filter>
 </service>
```
Add your custom service (which modifies twilio notification params) to the manifest:
```
<service
  android:name=".MainMessagingService"
  android:exported="false">
  	<intent-filter>
  		<action android:name="com.google.firebase.MESSAGING_EVENT" />
  	</intent-filter>
</service>
```

