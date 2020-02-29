
# HyperTrack Android SDK Integration Guide

[HyperTrack](https://www.hypertrack.com) lets you add live location tracking to your mobile app.
Live location is made available along with ongoing activity, tracking controls and tracking outage with reasons.
This document contains a step by step guide how to integrate HyperTrack SDK with your app within minutes.

## Create HyperTrack Account

[Sign up](https://dashboard.hypertrack.com/signup) for HyperTrack and
get your publishable key from the [Setup page](https://dashboard.hypertrack.com/setup).

### Add Hypertrack SDK
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
    implementation 'com.hypertrack:hypertrack:4.1.0'
    ...
}
```
### Set up silent push notifications

Set up silent push notifications to manage on-device tracking using HyperTrack cloud APIs from your server. This requires Firebase push notification. If you do not yet have push notifications enabled, please proceed to [setup Firebase Cloud Messaging](https://firebase.google.com/docs/android/setup).

Next step is to specify `HyperTrackMessagingService` as push messages receiver by adding the following snippet to your app's Android manifest:
```xml
...
  <service android:name="com.hypertrack.sdk.HyperTrackMessagingService" android:exported="false">
      <intent-filter>
          <action android:name="com.google.firebase.MESSAGING_EVENT" />
      </intent-filter>
  </service>
</application>
```
If you already use Firebase push notifications you can extend `HyperTrackMessagingService` instead of Firebase, or declare two receivers side by side, if you wish (don't forget to call `super.onNewToken` and `super.onMessageReceived` in that case).
Check out [Quickstart app](https://github.com/hypertrack/quickstart-android/) if you prefer to see an example.

The last step is to add your Firebase API key to [HyperTrack dashboard](https://dashboard.hypertrack.com/setup) under *Server to Device communication* section.

<aside> Push notifications have delays so if you're looking for more instant channel you can use <a href="https://hypertrack.github.io/sdk-android-hidden/javadoc/latest/com/hypertrack/sdk/HyperTrack.html#syncDeviceSettings--"><code>syncDeviceSettings</code></a> sdk method to speed up command propagatioon.</aside>

### Initialize SDK
Obtain an SDK instance, when you wish to use SDK, by passing your publishable key.
```java
  HyperTrack sdkInstance = HyperTrack.getInstance(MyActivity.this, "your-publishable-key-here");
```
```kotlin
  val sdkInstance = HyperTrack.getInstance(this, "your-publishable-key-here")
```
Also make sure you've requested permissions somewhere in your app. HyperTrack accesses location and activity data, so exact set of permissions depends on an Android version. You may use `HyperTrack.requestPermissionsIfNecessary()` convenience method to request permissions and make SDK integration simpler.

### Identify your devices

HyperTrack uses string device identifiers that could be obtained from the SDK instance
```java
   String deviceId = sdkInstance.getDeviceID();
```
```kotlin
   val deviceId = sdkInstance.deviceID
```

Make sure you've saved this device identifier as it is required when calling HyperTrack APIs.

## Start tracking

Now the app is ready to be tracked from the cloud. HyperTrack gives you powerful APIs
to control device tracking from your backend.

> To use the HyperTrack API, you will need the `{AccountId}` and `{SecretKey}` from the [Setup page](https://dashboard.hypertrack.com/setup).

### Track devices during work

Track devices when user is logged in to work, or during work hours by calling the
[Devices API](https://docs.hypertrack.com/#references-apis-devices).

To start, call the [start](https://docs.hypertrack.com/?shell#references-apis-devices-post-devices-device_id-start) API.

```
curl -X POST \
  -u {AccountId}:{SecretKey} \
  https://v3.api.hypertrack.com/devices/{device_id}/start
```


Get the tracking status of the device by calling
[GET /devices/{device_id}](https://docs.hypertrack.com/?shell#references-apis-devices-get-devices) api.

```
curl \
  -u {AccountId}:{SecretKey} \
  https://v3.api.hypertrack.com/devices/{device_id}
```

To see the device on a map, open the returned embed_url in your browser (no login required, so you can add embed these views directly to you web app).
The device will also show up in the device list in the [HyperTrack dashboard](https://dashboard.hypertrack.com/).

To stop tracking, call the [stop](https://docs.hypertrack.com/?shell#references-apis-devices-post-devices-device_id-stop) API.

```
curl -X POST \
  -u {AccountId}:{SecretKey} \
  https://v3.api.hypertrack.com/devices/{device_id}/stop
```

### Track trips with ETA

If you want to track a device on its way to a destination, call the [Trips API](https://docs.hypertrack.com/#references-apis-trips-post-trips)
and add destination.

HyperTrack Trips API offers extra fields to get additional intelligence over the Devices API.
* set destination to track route and ETA
* set scheduled_at to track delays
* share live tracking URL of the trip with customers
* embed live tracking view of the trip in your ops dashboard

```curl
curl -u {AccountId}:{SecretKey} --location --request POST 'https://v3.api.hypertrack.com/trips/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "device_id": "{device_id}",
    "destination": {
        "geometry": {
            "type": "Point",
            "coordinates": [{longitude}, {latitude}]
        }
    }
}'
```

To get `{longitude}` and `{latitude}` of your destination, you can use for example [Google Maps](https://support.google.com/maps/answer/18539?co=GENIE.Platform%3DDesktop&hl=en).

> HyperTrack uses [GeoJSON](https://en.wikipedia.org/wiki/GeoJSON). Please make sure you follow the correct ordering of longitude and latitude.

The returned JSON includes the embed_url for your dashboard and share_url for your customers.

When you are done tracking this trip, call [complete](https://docs.hypertrack.com/#references-apis-trips-post-trips-trip_id-complete) Trip API using the `trip_id` from the create trip call above.
```
curl -X POST \
  -u {AccountId}:{SecretKey} \
  https://v3.api.hypertrack.com/trips/{trip_id}/complete
```

After the trip is completed, use the [Trips API](https://docs.hypertrack.com/#references-apis-trips-post-trips) to
retrieve a full [summary](https://docs.hypertrack.com/#references-apis-trips-get-trips-trip_id-trip-summary) of the trip.
The summary contains the polyline of the trip, distance, duration and markers of the trip.

```
curl -X POST \
  -u {AccountId}:{SecretKey} \
  https://v3.api.hypertrack.com/trips/{trip_id}
```

### Track trips with geofences

If you want to track a device going to a list of places, call the [Trips API](https://docs.hypertrack.com/#references-apis-trips-post-trips)
and add geofences. This way you will get arrival, exit, time spent and route to geofences. Please checkout our [docs](https://docs.hypertrack.com/#references-apis-trips-post-trips) for more details.

## Dashboard

Once your app is running, go to the [dashboard](https://dashboard.hypertrack.com/devices) where you can see a list of all your devices and their live location with ongoing activity on the map.

## Advanced integration

###### Add SDK state listener to catch events.
You can subscribe to SDK status changes [`addTrackingListener`](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/HyperTrack.html#addTrackingListener-com.hypertrack.sdk.TrackingStateObserver.OnTrackingStateChangeListener-) and handle them in the appropriate methods [`onError(TrackingError)`](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onError-com.hypertrack.sdk.TrackingError-) [`onTrackingStart()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onTrackingStart--) [`onTrackingStop()`](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/TrackingStateObserver.OnTrackingStateChangeListener.html#onTrackingStop--)

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
Check out other configurable properties in [ServiceNotificationConfig reference](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/ServiceNotificationConfig.html)

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

Look into [documentation](https://hypertrack.github.io/sdk-android-hidden/javadoc/4.1.0/com/hypertrack/sdk/HyperTrack.html) for more details.

#### You are all set

You can now run the app and start using HyperTrack. You can see your devices on the [dashboard](#dashboard).
Also you can find more about SDK integration [here](https://github.com/hypertrack/live-app-android).

## Frequently Asked Questions
<details><summary><b id="faq-supported-api-levels">What API levels (Android versions) are supported</b></summary>
Currently we do support all of the Android versions starting from API 19 (Android 4.4 Kit Kat).
<p/>
</details>

<details><summary><b id="faq-no-class-def-found">NoClassDefFoundError</b></summary>

I've added SDK and my app started failing with message like `Fatal Exception: java.lang.NoClassDefFoundError`.
The reason of it, is that on Android API level 19 and below you cannot have more than 65536 methods in your app (including libraries methods). Please, check [this Stackoverflow](https://stackoverflow.com/questions/34997835/fatal-exception-java-lang-noclassdeffounderror-when-calling-static-method-in-an) answer for solutions.
<p/>
</details>

<details><summary><b id="faq-dependencies-conflict">Dependencies Conflicts</b></summary>
SDK dependencies graph looks like below:

````
+--- com.android.volley:volley:1.1.1
+--- com.google.code.gson:gson:2.8.6
+--- org.greenrobot:eventbus:3.1.1
+--- com.parse.bolts:bolts-tasks:1.4.0
+--- net.grandcentrix.tray:tray:0.12.0
+--- com.google.android.gms:play-services-location:15.0.1
|    +--- com.google.android.gms:play-services-base:[15.0.1,16.0.0) -> 15.0.1
|    |    +--- com.google.android.gms:play-services-basement:[15.0.1] -> 15.0.1
|    |    |    \--- com.android.support:support-v4:26.1.0
|    |    |         +--- com.android.support:support-compat:26.1.0
|    |    |         |    +--- com.android.support:support-annotations:26.1.0
|    |    |         |    \--- android.arch.lifecycle:runtime:1.0.0
|    |    |         |         +--- android.arch.lifecycle:common:1.0.0
|    |    |         |         \--- android.arch.core:common:1.0.0
|    |    |         +--- com.android.support:support-media-compat:26.1.0
|    |    |         |    +--- com.android.support:support-annotations:26.1.0
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)
|    |    |         +--- com.android.support:support-core-utils:26.1.0
|    |    |         |    +--- com.android.support:support-annotations:26.1.0
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)
|    |    |         +--- com.android.support:support-core-ui:26.1.0
|    |    |         |    +--- com.android.support:support-annotations:26.1.0
|    |    |         |    \--- com.android.support:support-compat:26.1.0 (*)
|    |    |         \--- com.android.support:support-fragment:26.1.0
|    |    |              +--- com.android.support:support-compat:26.1.0 (*)
|    |    |              +--- com.android.support:support-core-ui:26.1.0 (*)
|    |    |              \--- com.android.support:support-core-utils:26.1.0 (*)
|    |    \--- com.google.android.gms:play-services-tasks:[15.0.1] -> 15.0.1
|    |         \--- com.google.android.gms:play-services-basement:[15.0.1] -> 15.0.1 (*)
|    +--- com.google.android.gms:play-services-basement:[15.0.1,16.0.0) -> 15.0.1 (*)
|    +--- com.google.android.gms:play-services-places-placereport:[15.0.1,16.0.0) -> 15.0.1
|    |    \--- com.google.android.gms:play-services-basement:[15.0.1,16.0.0) -> 15.0.1 (*)
|    \--- com.google.android.gms:play-services-tasks:[15.0.1,16.0.0) -> 15.0.1 (*)
+--- com.android.support:support-annotations:26.1.0
+--- com.google.firebase:firebase-messaging:17.1.0
|    +--- com.google.android.gms:play-services-basement:15.0.1 (*)
|    +--- com.google.android.gms:play-services-tasks:15.0.1 (*)
|    +--- com.google.firebase:firebase-common:16.0.0
|    |    +--- com.google.android.gms:play-services-basement:15.0.1 (*)
|    |    \--- com.google.android.gms:play-services-tasks:15.0.1 (*)
|    +--- com.google.firebase:firebase-iid:[16.2.0] -> 16.2.0
|    |    +--- com.google.android.gms:play-services-basement:15.0.1 (*)
|    |    +--- com.google.android.gms:play-services-stats:15.0.1
|    |    |    \--- com.google.android.gms:play-services-basement:[15.0.1] -> 15.0.1 (*)
|    |    +--- com.google.android.gms:play-services-tasks:15.0.1 (*)
|    |    +--- com.google.firebase:firebase-common:16.0.0 (*)
|    |    \--- com.google.firebase:firebase-iid-interop:16.0.0
|    |         +--- com.google.android.gms:play-services-base:15.0.1 (*)
|    |         \--- com.google.android.gms:play-services-basement:15.0.1 (*)
|    \--- com.google.firebase:firebase-measurement-connector:16.0.0
|         \--- com.google.android.gms:play-services-basement:15.0.1 (*)
````

Common problem here is depending on different versions of `com.android.support` library components. You can explicitly specify required version by adding it as a dependency in your app's `build.gradle`, e.g.:

````groovy
implementation `com.android.support:support-v4:28.0.0`
````

and explicitly force SDK pick app's dependencies

```
implementation("com.hypertrack:hypertrack:4.1.0") {transitive = false}
```

That will take precedence over SDK version and you'll have one version of support library on your classpath.
<p/>
</details>

<details><summary><b id="faq-persistent-notification">Persistent notification</b></summary>

HyperTrack SDK, by default, runs as a foreground service. This is to ensure that the location tracking works reliably even when your app is minimized. A foreground service is a service that the user is actively aware of and isn't a candidate for the system to kill when low on memory.
Android mandates that a foreground service provides a persistent notification in the status bar. This means that the notification cannot be dismissed by the user.

![persistent-notification](https://user-images.githubusercontent.com/10487613/54007190-6ec47c00-4115-11e9-9743-332befbcf8f5.png)
<p/>
</details>

<details><summary><b id="faq-handling-custom-roms">Handling custom ROMs</b></summary>

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

<details><summary><b id="faq-persistent-notification">HyperTrack notification shows even after app is terminated</b></summary>

The HyperTrack service runs as a separate component and it is still running when the app that started it is terminated. That is why you can observe that notification. When you tracking is stopped, the notification goes away.
<p/>
</details>

<details><summary><b id="faq-doze-mode">How tracking works in Doze mode</b></summary>

Doze mode requires device [to be stationary](https://developer.android.com/training/monitoring-device-state/doze-standby.html#understand_doze), so before OS starts imposing power management restrictions, exact device location is obtained. When device starts moving, Android leaves Doze mode and works regularly, so no special handling of Doze mode required with respect to location tracking.
<p/>
</details>

<details><summary><b id="faq-aapt-error-foreground-service-type">AAPT: error: attribute android:foregroundServiceType not found</b></summary>

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
