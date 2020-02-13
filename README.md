
# HyperTrack Android SDK Integration Guide

[HyperTrack](https://www.hypertrack.com) lets you add live location tracking to your mobile app. This doc contains an step by step guide how to make it works.

* [Publishable Key](#publishable-key) – Get your Publishable Key
* [Basic integration](#basic-integration) – Basic Integration
* [Advanced integration](#advanced-integration) – When Basic is not enough
* [FAQs](#frequently-asked-questions) – Frequently asked questions

## Publishable Key

We use Publishable Key to identify your devices. To get one:
1. Go to the [Signup page](https://dashboard.hypertrack.com/signup). Enter your email address and password.
2. Open the verification link sent to your email.
3. Open the [Setup page](https://dashboard.hypertrack.com/setup), where you can copy your Publishable Key.


## Basic integration

 - [Add Hypertrack SDK](#step-1-add-hypertrack-sdk)
 - [Enable server to device communication](#step-2-enable-server-to-device-communication)
 - [Initialize SDK](#step-3-initialize-sdk)
 - [Identify your device](#step-4-identify-your-devices)
 - [Verify integration](#step-5-verify-your-integration)

#### Step 1. Add Hypertrack SDK
Add following lines to your applications `build.gradle`:
```
// Import the SDK within your repositories block
repositories {
    maven {
        name 'hypertrack'
        url  'https://s3-us-west-2.amazonaws.com/m2.hypertrack.com/'
    }
    ...
}

//Add HyperTrack as a dependency
dependencies {
    implementation 'com.hypertrack:hypertrack:4.0.0-SNAPSHOT'
    ...
}
```

#### Step 2. Enable server to device communication.
Server to device communication uses firebase push notifications as transport for commands, so for remote tracking state management Firebase integration is required. So you need to [setup Firebase Cloud Messaging](https://firebase.google.com/docs/android/setup), if you have no push notifications enabled so far. Next step is to specify `HyperTrackMessagingService` as push messages receiver by adding following snippet to your apps Android manifest:
```xml
...
  <service android:name="com.hypertrack.sdk.HyperTrackMessagingService" android:exported="false">
      <intent-filter>
          <action android:name="com.google.firebase.MESSAGING_EVENT" />
      </intent-filter>
  </service>
</application>
```
If you already use firebase push notifications you can extend `HyperTrackMessagingService` instead of Firebase, or declare two receivers side by side, if you wish (don't forget to call `super.onNewToken` and `super.onMessageReceived` in that case).
Check out [Quickstart app](https://github.com/hypertrack/quickstart-android/) if you prefer to get a look at example.
Last step is to add your Firebase API key to [HyperTrack dashboard](https://dashboard.hypertrack.com/setup) under *Server to Device communication* section.

#### Step 3. Initialize SDK
Retrieve sdk instance, when you wish to use SDK, by passing your [publishable key]().
```java
  HyperTrack sdkInstance = HyperTrack.getInstance(MyActivity.this, "your-publishable-key-here");
```
```kotlin
  val sdkInstance = HyperTrack.getInstance(this, "your-publishable-key-here")
```
Also make sure you've requested permissions somewhere in your app. HyperTrack accesses location and activity data, so exact set of permissions depends on Android version, and you can use `HyperTrack.requestPermissionsIfNecessary()` convenience method to simplify it a bit.

#### Step 4. Identify your devices

HyperTrack uses string device identifiers that could be obtained from sdk instance
```java
   String deviceId = sdkInstance.getDeviceID();
```
```kotlin
   val deviceId = sdkInstance.deviceID
```

Make sure you've saved this identifier as it is required when calling HyperTrack APIs.

#### Step 5. Verify your Integration

Once you've finished pervious steps you are ready to test your integration.
Simplest way to start tracking is to use [HyperTrack public API](https://docs.hypertrack.com/#references-apis-devices-post-devices-device_id-start) and you need to copy your authentication credentials from [Dashboard](https://dashboard.hypertrack.com/setup), so the start command will look like below:
```bash
curl https://v3.api.hypertrack.com/devices/{deviceId}/start \
    --request POST \
    --user {AccountId}:{SecretKey} \
```

Once the command above is done, you'll see tracking notification on device and will be able to see its location on dashboard.

When you're done you can finish the trip using following bash command:
```bash
curl https://v3.api.hypertrack.com/devices/{deviceId}/stop \
    --request POST \
    --user {AccountId}:{SecretKey} \
```



## Advanced integration
###### Create trip marker
Use this optional method if you want to associate data with specific place in your trip. E.g. user marking a task as done, user tapping a button to share location, user accepting an assigned job, device entering a geofence, etc.
```java
HyperTrack sdkInstance = HyperTrack.getInstance(context, publishableKey);

Map<String, Object> order = new HashMap<>();
order.put("item", "Martin D-18");
order.put("previousOwners", Collections.emptyList());
order.put("price", 7.75);

sdkInstance.addTripMarker(order);
```

Look into [documentation](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/HyperTrack.html) for more details.

#### Step 3. _(optional)_ Utility Methods
###### Turn tracking on and off
Depending on your needs, you can always _stop_ and _start_ tracking, invoking [`.stop()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/HyperTrack.html#stop--) and [`start()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/HyperTrack.html#start--) SDK methods.
It is recommended to store reference to SDK instance in order to use it for further actions. You can determine current sdk state using [`isRunning()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/HyperTrack.html#isRunning--) call.

###### Add SDK state listener to catch events.
You can subscribe to SDK status changes [`addTrackingListener`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/HyperTrack.html#addTrackingListener-com.hypertrack.sdk.TrackingStateObserver.OnTrackingStateChangeListener-) and handle them in the appropriate methods [`onError(TrackingError)`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onError-com.hypertrack.sdk.TrackingError-) [`onTrackingStart()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onTrackingStart--) [`onTrackingStop()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onTrackingStop--)

###### Customize foreground service notification
HyperTrack tracking runs as a separate foreground service, so when it is running, your users will see a persistent notification. By default, it displays your app icon with text `{app name} is running` but you can customize it anytime after initialization by calling:
```java
HyperTrack sdkInstance = HyperTrack.getInstance(context, publishableKey);
sdkInstance.setTrackingNotificationConfig(
                new ServiceNotificationConfig.Builder()
                        .setContentTitle("Tap to stop tracking")
                        .build()
        );
```
Check out other configurable properties in [ServiceNotificationConfig reference](https://hypertrack.github.io/sdk-android-hidden/javadoc/3.8.3/com/hypertrack/sdk/ServiceNotificationConfig.html)

#### You are all set

You can now run the app and start using HyperTrack. You can see your devices on the [dashboard](#dashboard).
Also you can find more about SDK integration [here](https://github.com/hypertrack/live-app-android).

## Frequently Asked Questions
<details><summary><b>What API levels (Android versions) are supported</b></summary>
Currently we do support all of the Android versions starting from API 19 (Android 4.4 Kit Kat).
<p/>
</details>

<details><summary><b>NoClassDefFoundError</b></summary>

I've added SDK and my app started failing with message like `Fatal Exception: java.lang.NoClassDefFoundError`.
The reason of it, is that on Android API level 19 and below you cannot have more than 65536 methods in your app (including libraries methods). Please, check [this Stackoverflow](https://stackoverflow.com/questions/34997835/fatal-exception-java-lang-noclassdeffounderror-when-calling-static-method-in-an) answer for solutions.
<p/>
</details>

<details><summary><b>Dependencies Conflicts</b></summary>
SDK dependencies graph looks like below:

````
+--- com.android.volley:volley:1.1.0<br/>
+--- com.google.code.gson:gson:2.8.5<br/>
+--- org.greenrobot:eventbus:3.1.1<br/>
+--- com.parse.bolts:bolts-tasks:1.4.0<br/>
+--- net.grandcentrix.tray:tray:0.12.0<br/>
|    \--- com.android.support:support-annotations:23.0.1 -> 28.0.0<br/>
+--- com.google.android.gms:play-services-location:16.0.0<br/>
|    +--- com.google.android.gms:play-services-base:16.0.1<br/>
|    |    +--- com.google.android.gms:play-services-basement:16.0.1<br/>
|    |    |    \--- com.android.support:support-v4:26.1.0<br/>
|    |    |         +--- com.android.support:support-compat:26.1.0<br/>
|    |    |         |    +--- com.android.support:support-annotations:26.1.0 -> 28.0.0<br/>
|    |    |         |    \--- android.arch.lifecycle:runtime:1.0.0<br/>
|    |    |         |         +--- android.arch.lifecycle:common:1.0.0<br/>
|    |    |         |         \--- android.arch.core:common:1.0.0<br/>
|    |    |         +--- com.android.support:support-media-compat:26.1.0<br/>
|    |    |         |    +--- com.android.support:support-annotations:26.1.0 -> 28.0.0<br/>
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)<br/>
|    |    |         +--- com.android.support:support-core-utils:26.1.0<br/>
|    |    |         |    +--- com.android.support:support-annotations:26.1.0 -> 28.0.0<br/>
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)<br/>
|    |    |         +--- com.android.support:support-core-ui:26.1.0<br/>
|    |    |         |    +--- com.android.support:support-annotations:26.1.0 -> 28.0.0<br/>
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)<br/>
|    |    |         \--- com.android.support:support-fragment:26.1.0<br/>
|    |    |              +--- com.android.support:support-compat:26.1.0 (*)<br/>
|    |    |              +--- com.android.support:support-core-ui:26.1.0 (*)<br/>
|    |    |              \--- com.android.support:support-core-utils:26.1.0 (*)<br/>
|    |    \--- com.google.android.gms:play-services-tasks:16.0.1<br/>
|    |         \--- com.google.android.gms:play-services-basement:16.0.1 (*)<br/>
|    +--- com.google.android.gms:play-services-basement:16.0.1 (*)<br/>
|    +--- com.google.android.gms:play-services-places-placereport:16.0.0<br/>
|    |    \--- com.google.android.gms:play-services-basement:16.0.1 (*)<br/>
|    \--- com.google.android.gms:play-services-tasks:16.0.1 (*)<br/>
\--- com.android.support:support-annotations:28.0.0<br/>
````

Common problem here is depending on different versions of `com.android.support` library components. You can explicitly specify required version by adding it as a dependency in your app's `build.gradle`, e.g.:

````groovy
implementation `com.android.support:support-v4:28.0.0`
````

That will take precedence over SDK version and you'll have one version of support library on your classpath.
<p/>
</details>

<details><summary><b>Persistent notification</b></summary>

HyperTrack SDK, by default, runs as a foreground service. This is to ensure that the location tracking works reliably even when your app is minimized. A foreground service is a service that the user is actively aware of and isn't a candidate for the system to kill when low on memory.
Android mandates that a foreground service provides a persistent notification in the status bar. This means that the notification cannot be dismissed by the user.

![persistent-notification](https://user-images.githubusercontent.com/10487613/54007190-6ec47c00-4115-11e9-9743-332befbcf8f5.png)
<p/>
</details>

<details><summary><b>Handling custom ROMs</b></summary>

Smartphones are getting more and more powerful, but the battery capacity is lagging behind. Device manufactures are always trying to squeeze some battery saving features into the firmware with each new Android release. Manufactures like Xiaomi, Huawei and OnePlus have their own battery savers that kills the services running in the background.
To avoid OS killing the service, users of your app need to override the automatic battery management and set it manual. To inform your users and direct them to the right setting page, you may add the following code in your app. This would intent out your user to the right settings page on the device.

````java
try {
  Intent intent = new Intent();
  String manufacturer = android.os.Build.MANUFACTURER;
  if ("xiaomi".equalsIgnoreCase(manufacturer)) {
    intent.setComponent(new ComponentName("com.miui.securitycenter", "com.miui.permcenter.autostart.AutoStartManagementActivity"));
  }
  else if ("oppo".equalsIgnoreCase(manufacturer)) {
    intent.setComponent(new ComponentName("com.coloros.safecenter", "com.coloros.safecenter.permission.startup.StartupAppListActivity"));
  }
  else if ("vivo".equalsIgnoreCase(manufacturer)) {
    intent.setComponent(new ComponentName("com.vivo.permissionmanager", "com.vivo.permissionmanager.activity.BgStartUpManagerActivity"));
  }

  List<ResolveInfo> list = context.getPackageManager().queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
  if  (list.size() > 0) {
    context.startActivity(intent);
  }
}
catch (Exception e) {
  Crashlytics.logException(e);
}
````

You may also try out open source libraries like [AutoStarter](https://github.com/judemanutd/AutoStarter).

Some manufacturers don't allow to whitelist apps programmatically. In that case the only way to achieve service reliability is manual setup. E.g. for Oxygen OS (OnePlus) you need to select *Lock* menu item from app options button in _Recent Apps_ view:

![one-plus-example](https://user-images.githubusercontent.com/10487613/58070846-388a8a80-7ba3-11e9-8b4f-11e39d26382b.png)
<p/>
</details>

<details><summary><b>HyperTrack notification shows even after app is terminated</b></summary>

The HyperTrack service runs as a separate component and it is still running when the app that started it is terminated. That is why you can observe that notification. When you tracking is stopped, the notification goes away.
<p/>
</details>

<details><summary><b>How tracking works in Doze mode</b></summary>

Doze mode requires device [to be stationary](https://developer.android.com/training/monitoring-device-state/doze-standby.html#understand_doze), so before OS starts imposing power management restrictions, exact device location is obtained. When device starts moving, Android leaves Doze mode and works regularly, so no special handling of Doze mode required with respect to location tracking.
<p/>
</details>

<details><summary><b>AAPT: error: attribute android:foregroundServiceType not found</b></summary>

If build fails with error like `AAPT: error: attribute android:foregroundServiceType not found` that means that you're targeting your app for Android P or earlier. Starting from Android 10 Google imposes additional restrictions on services, that access location data while phone screen is turned off. Possible workaround here is to remove declared service property by adding following element to your app's manifest
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.hypertrack.quickstart"
  xmlns:tools="http://schemas.android.com/tools">
  ...
  <application>
    ...
    <service android:name="com.hypertrack.sdk.service.HyperTrackSDKService"
      tools:remove="android:foregroundServiceType" />
```

Although you'll be able to avoid targeting API 29, but tracking service won't work properly on Android Q devices with screen been turned off or locked.
<p/>
</details>

## Support
Join our [Slack community](https://join.slack.com/t/hypertracksupport/shared_invite/enQtNDA0MDYxMzY1MDMxLTdmNDQ1ZDA1MTQxOTU2NTgwZTNiMzUyZDk0OThlMmJkNmE0ZGI2NGY2ZGRhYjY0Yzc0NTJlZWY2ZmE5ZTA2NjI) for instant responses. You can also email us at help@hypertrack.com.
