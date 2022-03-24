# Changelog

## [6.0.2] - 2022-03-18
### Changed
- The project that uses Hypertrack SDK can be compiled with Android SDK API 30

## [6.0.1] - 2022-03-01
### Fixed
- Crash in SDK internals

## [6.0.0] - 2022-02-23
### Added
- Added GPS outage detection.
- Added battery status reporting. You can check the battery level in our APIs.
- Added robust location spoofing detection. Any location spoofing attempt is reported in our APIs and dashboards.
### Changed
- Improved tracking quality. Now the movement is captured with greater fidelity. This in turn improves visit detection and distance calculations.
- Improved time to the first location.
- Improved battery life. SDK now tracks less frequently when there is less movement, which conserves battery life.
- Improved location mocking in development mode. Use `allowMockLocations()` API in conjunction with disabling cellular/Bluetooth/WiFi assistance in Settings > Location to test how your app reacts to mocked movement.
- Improved activity recognition detection.
- Improved configurability of the SDK.
### Fixed
- Fixed getLatestLocation API, now it properly returns the latest location sent to the cloud.
- Fixed automatic config updates in runtime.
- Fixed step detection bugs.
- Fixed ANR triggers.

## [5.4.5] - 2021-11-05
### Fixed
- Added FLAG_MUTABLE to PendingIntents to avoid crashes on Android 12

## [5.4.4] - 2021-10-29
### Fixed
- Crash related to notification icon on some Android 11 devices

## [5.4.3] - 2021-09-01
### Fixed
- SDK_KILLED outage description (added "description" field to health event)

## [5.4.2] - 2021-08-31
### Fixed
- Critical bug in whitelisting prompt

## [5.4.1] - 2021-08-30
### Added
- Battery data in location events
- Whitelisting instructions for more device manufacturers
- SDK_KILLED outage description

## [5.4.0] - 2021-08-04
### Added
- Mock location outage
### Removed
- Sync on app going to background and foreground

## [5.3.0] - 2021-07-16
### Added
- start/stop commands are stacked to queue and processed one-by-one to prevent race conditions.
### Fixed
- SDK now can start tracking if Firebase isn't enabled for the app.

## [5.2.5] - 2021-07-02
### Changed
- Added explicit check to ensure we do not add lifecycle observer on the background thread
(we relied on [framework guarantees before](https://developer.android.com/reference/kotlin/android/content/ContentProvider?hl=en#onCreate()))
- Ignored start/stop commands are reported as status updates. No concealing on permission change.

## [5.2.4] - 2021-06-17
### Fixed
- Cached location has timestamp that was previously missing. Now it can be examined, if that's
matters. We still invalidate locations based on accelerometer data when we believe they are no
longer relevant.
- Permission denial on a first run wasn't reported on Android 11 (now fixed).
- Fixed caching issue, that could lead to `isRunning` returns true with service been stopped.
- Added a separate button to the whitelistening prompt. Now it's possible to review the steps,
since the dialog remains visible until *Done* is clicked. The behavior is also changed, since the
state resets back to default when application was terminated by OS with meaningless reason.

<p align="center">
 <img src="https://user-images.githubusercontent.com/10487613/122400240-35317b00-cf84-11eb-8787-1910fbec8af3.gif" width="25%" height="25%" />
</p>

## [5.2.3] - 2021-06-15
### Fixed
- SDK now sends outage start timestamp when restarted by push notification.

## [5.2.2] - 2021-06-11
### Fixed
- Disk access from the main thread on Android 11 devices
### Changed
- Switched from range dependency versions declaration to a single version one. Google Services
plugin doesn't work with range version, so this change is intended to ease the integration effort
by compromising explicit dependency version specification.

## [5.2.1] - 2021-06-10
### Fixed
- Automatic tracking restart on device reboot wasn't working on Android 11 but it's fixed now.
Background location access permission is required for this flow to work.

## [5.2.0] - 2021-06-07
### Added
- Outage resolution API. Use `HyperTrack.getBlockers()` to get the tracking impediments details.
### Fixed
- Removed push token caching to resist bug in [firebase-sdk v20.1.1](https://firebase.google.com/support/release-notes/android#messaging_v20-1-1)
- Sync service (internal) bug fixed

## [5.1.0] - 2021-05-28
### Added
- Compatibility with Firebase Messaging extended to be from 17.0.0 to 22.0.0
- Methods to retrieve current (async) and the latest known (blocking) location.
### Fixed
- NPE crash in Gson trying to deserialize empty collection

## [5.0.0] - 2021-05-23
### Added
- Geotag method returns current device location or the reason, why it can't be retrieved
### Removed
- Restricted Geotag method removed

## [4.13.0] - 2021-05-19
### Added
- Android Strict mode compliance
### Fixed
- Crash that occurred on network response been delivered after app context is invalidated

## [4.12.2] - 2021-05-11
### Fixed
- Main thread API contract violation bug fixed.

## [4.12.1] - 2021-05-10
### Fixed
- Crash on concurrent start from multiple threads was fixed.

## [4.12.0] - 2021-05-07
### Added
- Whitelisting hint will appear on devices, that are known to have additional permissions (Huawei, Samsung, Realme etc.). `sdk.requestPermissionsIfNecessary()` or `sdk.start()` trigger that prompt.
### Fixed
- HyperTrackSDKService lifecycle improved. `startForeground` is called immediately, without any disk access operations.

## [4.11.1] - 2021-04-20
### Fixed
- Bug when SDK didn't inform about GPS signal unavailability, if it was unavailable from the tracking start.
- Unknown outages won't be reported like restarted by user.

## [4.11.0] - 2021-03-30
### Added
- Restricted geotag interface method, that creates geotag when device is within specified region and fails otherwise.

## [4.10.1] - 2021-03-19
### Fixed
- Fixed concurrency bug resulted in multiple outages being genrated

## [4.10.0] - 2021-02-22
### Added
- Dynamic publishable key change support (comes in handy for test/prod switching)
- Background Location permission was made optional (with a dedicated setter on SDK instance)
### Fixed
- Handler based scheduler replaced with coroutines as a workaround to a memory leek in OS APIs
- Fixed race conditions that could causes involuntary tracking stops
- Human readable explanation of process exit reason on Android 11
- Constant device id (was only on Oreo or later before)

## [4.9.0] - 2020-12-23
### Fixed
- `HypeTrackMessagingService` was removed to avoid requirement of overriding it instead of `FirebaseMessagingService`.

## [4.8.0] - 2020-10-30
### Fixed
- Backend and local tracking state conflict fix via their timestamps comparison.
- Sparse locations after exits from stops are no longer appears.

## [4.7.0] - 2020-10-16
### Added
- Android 11 compartibility: SDK will ask for background location access permission on Android 11
and correctly detect location access restrictions, that caused by new permission policy.
### Fixed
- Fixed a bug when SDK had delay in increasing locations frequency after stops.
- Removed default large icon in tracking notification as per Material Design Guidelines
- Small icon is now also configurable through overridden resources.
Vector drawable with name `ic_hypertrack_notification` will be used as a default small notifiation icon.

## [4.6.0] - 2020-09-25
### Added
- Automatic sync on internal triggers (sdk init, publishable key set etc).
- SDK dynamic configuration from remote.
### Fixed
- Steps counter reporting total instead of increment
- Invalid locations client-side check

## [4.5.4] - 2020-09-21
### Fixed
- Fixed a database migration bug that could result in inability of creating geotags

## [4.5.3] - 2020-07-27
### Fixed
- Bug that blocked metadata propagation from device to platform on Android 4.4 (API 19).
 
## [4.5.2] - 2020-07-24
### Changed
- Stability: switched from Parceable serialization to avoid deserialization issues on reboot.
 
## [4.5.1] - 2020-07-20
### Changed
- Device name and metadata changes are propagated to platform immediately.
- Missing permission error is only reported on tracking start.

## [4.5.0] - 2020-07-13
### Added
- Added logic to mark tracking segments where location updates weren't available.


## [4.4.1] - 2020-06-16
### Changed
- Custom markers were renamed to geotags

## [4.4.0] - 2020-06-16
### Added
- Expected location in custom Marker
- Enabled network payload compression

## [4.3.2] - 2020-05-10
### Changed
- Events batching on stops was disabled in fake location mode to ease testing

## [4.3.1] - 2020-05-05
### Added
- Added HyperTrackMessaging service to sdk manifest to ease the integration process.

## [4.3.0] - 2020-04-28
### Changed
- Custom markers (formerly known as trip markers or custom events) were made propagating errors on invalid payload, instead of throwing errors.
- Crash on notification click with unserializable Pending Intent fixed.
- Minor changes

## [4.1.3] - 2020-03-23
### Added
- Added changes to device 2 platform contract to improve disconnected devices detection consistency

## [4.1.2] - 2020-03-13
### Changed
- End of trial period handling logic changed to avoid unnecessary networking (retries etc.)
- Fixed a crash due to uninformative exception in underlying SQLite Db initialization call.

## [4.1.0] - 2020-02-29
### Changed
- Switched from grandcentrix's tray library to custom local sqlite storage implementation
- Firebase dependency version specified as required (was strict before)

## [4.0.0] - 2020-02-25
### Added
- Notification about 403 authentication error is propagated to SDK state observer.
### Removed
- Removed possibility to create a trip for this device directly from the sdk.

## [3.8.5] - 2020-02-16
### Changed
- Fixed recursion bug that led to battery drain

## [3.8.4] - 2020-02-14
### Changed
- Remote tracking intent is distinguished from sdk API intent and visible in dashboard

## [3.8.3] - 2020-01-24
### Changed
- Fixed a crash due to onNewToken call outside of looped thread
- Fixed a crash due to broken lifecycle invariants

## [3.8.0] - 2020-01-08
### Added
- Added possibility to create a trip for the device directly from SDK.
### Changed
 - Android 10 background data access issue fixed

## [3.7.0] - 2019-12-20
### Changed
- SDK startup time reduced due to programmatic detection of auth token expiration

## [3.6.0] - 2019-12-03
### Changed
- Excessive wake lock usage eliminated via switching from `AlarmManager` to `Handler` based tasks scheduling.

## [3.5.2] - 2019-11-14
### Changed
- Null device metadata bug fixed

## [3.5.0] - 2019-10-14
### Added
- New, instance centric, interface for SDK (old methods are kept, but marked deprecated).
- Possibility to configure persistent notification via config

## [3.4.6] - 2019-09-16
### Added
- Android 10 support

## [3.3.2] - 2019-08-13
### Changed
- Device ID generator changed to have publishable key among seeds, so device ids will be different
for the same physical device if sdk is initialized with different publishable Key (addresses
dev/prod credentials usage on the same device).

## [3.3.1] - 2019-08-08
### Changed
- Updated push notification feature to support token fetch during integration in app, that already received it from Firebase

## [3.4.3] - 2019-08-05
### Changed
- Fixed bugs in tracking observer functionality
- `isTracking` bug fixed.

## [3.4.2] - 2019-07-31
### Added
- Possibility to attach an observer, that is notified on SDK state changes
### Changed
- Displacement sensivity increased to finer tracking

## [3.3.0] - 2019-07-17
### Changed
- Trip markers replaced custom events.

## [3.2.0] - 2019-07-04
### Added
- Server to device communication support added. It is possible to start/stop tracking from platform.






