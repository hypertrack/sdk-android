# Plugins

## Introduction

HyperTrack SDK uses the **Plugin architecture** to be able to adapt to different dependencies and add configurable logic.

To use the SDK you have to add the core SDK dependency

```Groovy 
implementation "com.hypertrack:sdk-android:<version>"
```

And a set of plugins (the version is the same for the core SDK and all plugins)

```Groovy
implementation "com.hypertrack:location-services-google:<version>"
implementation "com.hypertrack:push-service-firebase:<version>"
```

**⚠️ For the SDK to work you have to include at least one plugin of each of those types:**
- Location Services
- Push Service

The Activity Service plugin is optional. It adds motion and activity detection when the app requests the Physical activity (`ACTIVITY_RECOGNITION`) permission and the user grants it.

## Available plugins

| Plugin Type | Maven Artifact ID | Minimal HyperTrack SDK version | Description |
| --- | --- | --- | --- |
| Location Services | `location-services-google` | 7.0.0 | Default Google Play Services Location API (depends on the newest version of `com.google.android.gms:play-services-location` at the time of plugin version release) |
| Location Services | `location-services-google-19-0-1` | 7.0.1 | Legacy Google Play Services Location API (uses version 19.0.1 of `com.google.android.gms:play-services-location`). It is useful when there is other dependency on pre-`20.0.0` Google Play Location library (more on this issue in the [Android SDK FAQ](https://hypertrack.com/docs/android-sdk-faq)) |
| Push Service | `push-service-firebase` | 7.0.0 | Default Firebase Cloud Messaging (depends the newest version of `com.google.firebase:firebase-messaging` at the time of plugin version release) |
| Activity Service | `activity-service-google` | 7.8.0 | Optional motion and activity detection using Google Play Services Activity Recognition (declares the `ACTIVITY_RECOGNITION` permission) |
