---
sidebar_position: 4
title: ❓ FAQ
description: Frequently asked questions.
---

If you have questions not answered here, please ask! Both filing an issue
or asking on [Discord](https://discord.gg/9hKJcWGcaB)) work.

### What is "code push"?

Code push, also referred to as "over the air updates" (OTA) is a cloud service
we are building to enable Flutter developers to deploy updates directly to
devices anywhere they have shipped Flutter. We've currently shown demos using
Android, but since it's built with Flutter/Dart (and Rust) it can eventually
work anywhere Flutter works.

The easiest way to see is to watch the [demo
video](https://www.youtube.com/watch?v=mmKvs0_Zu14&ab_channel=Shorebird).

"Code Push" is a reference to the name of a deploy feature used by the React
Native community from [Microsoft](https://appcenter.ms) and
[Expo](https://expo.dev), neither of which support Flutter.

### What is the roadmap?

We try to keep: https://docs.shorebird.dev/status up to date with the status
of the project.

Our project boards are also public an found at:
https://github.com/orgs/shorebirdtech/projects

### How does this relate to Firebase Remote Config or Launch Darkly?

Code push allows adding new code / replacing code on the device. Firebase
Remote Config and Launch Darkly are both configuration systems. They allow you
to change the configuration of your app without having to ship a new version.
They are not intended to replace code.

### How big of a dependency footprint does this add?

I haven't measured recently, but I expect the code push library to add less than
one megabyte to Flutter apps. `flutter build apk --release` vs.
`shorebird build apk --release` should give you a rough idea. We know of ways
we can make this smaller when that becomes a priority. If size is a blocker
for you, please let us know!

### How does this relate to Flutter Hot Reload?

Flutter's Hot reload is a development-time-only feature. Code push is for
production.

Hot reload is a feature of Flutter that allows you to change code on the device
during development. It requires building the Flutter engine with a debug-mode
Dart VM which includes a just-in-time (JIT) Dart compiler.

Code push is a feature that allows you to change code on the device in
production. We will use a variety of different techniques to make this possible
depending on the platform. Current demos execute ahead-of-time compiled Dart
code and do not require a JIT Dart compiler.

### What types of changes does Shorebird code push support?

Code push only supports changing Dart code at this time. We have plans to
support changing Flutter asset files (from pubspec.yaml) in the near future:
https://github.com/shorebirdtech/shorebird/issues/318

We do not have plans to support changing
native code (e.g. Java/Kotlin on Android or Objective-C/Swift on iOS), but we
do have plans to warn about native code changes during patch creation:
https://github.com/shorebirdtech/shorebird/issues/385

### Does this support Flutter Web?

Code push isn't needed for Flutter web as the web already works this way. When
a user opens a web app it downloads the latest version from the server if
needed.

If you have a use case for code push with Fluter web, we'd
[love to know](https://github.com/shorebirdtech/shorebird/issues/new?assignees=&labels=feature&template=feature_request.md&title=feat%3A+)!

### Will this work on iOS, Android, Mac, Windows, Linux, etc?

Yes.

So far we've focused on Android support, but code push will eventually work
everywhere Flutter works. We're ensuring we've built all the infrastructure
needed to provide code push reliably, safely first before expanding to more
platforms.

There are different technical restrictions on some platforms (e.g. iOS) relative
to Android, but we have several approaches we can use to solve them (Dart is an
incredibly flexible language).

### How does this relate to the App/Play Store review process or policies?

Developers are bound by their agreements with store providers when they choose
to use those stores. Code push is designed to allow developers to update their
apps and still comply with store policies on iOS and Android. Similar to the
variety of commercial products available to do so with React Native (e.g.
[Microsoft](https://appcenter.ms), [Expo](https://expo.dev)).

We will include more instructions and guidelines about how to both use code push
and comply with store guidelines as we get closer to a public launch.

### Does code push require the internet to work?

Yes. One could imagine running a server to distribute the updates separately
from the general internet, but some form of network connectivity is required to
transport updates to the devices.

### What happens if a user doesn't update for a long time and misses an update?

Our implementation always sends an update specifically tailored for the device
that is requesting it updating the requestor always to the latest version
available. Thus if a user doesn't update for a while they will "miss"
intermediate updates.

The update server could be changed to support responding with either the next
incremental version or the latest version depending on your application's needs.
Please let us know if alternative update behaviors are important to you.

### Why are some parts of the code push library written in Rust?

Parts of the code push ("updater") system are written in Rust:

1. Avoids starting two Dart VMs (one for the updater and one for the app).
2. Allows accessing the updater code from multiple languages (e.g. both the C++
   engine as well as a Dart/Flutter application, or even Kotlin/Swift code if
   needed)

See our [Languages
Philosophy](https://github.com/shorebirdtech/handbook/blob/main/engineering.md#languages)
for more information as to why we chose Rust.

## How does Shorebird relate to Flutter?

Shorebird is a fork of Flutter that adds code push. Shorebird is not a
replacement for Flutter, but rather a replacement for the Flutter engine. You
can continue to use the Flutter tooling you already know and love.

`shorebird` uses a fork of Flutter that includes the Shorebird updater. We track
the latest stable release of Flutter and replace a few of the Flutter engine
files with our modified copies.

To implement our fork, we use `FLUTTER_STORAGE_BASE_URL` to point to
`https://download.shorebird.dev` instead of download.flutter.dev. We pass
through unmodified output from the `flutter` tool so you will see a warning from
Flutter:

```
Flutter assets will be downloaded from http://download.shorebird.dev. Make sure you trust this source!
```

For more information about why we had to fork Flutter see
[architecture.md](architecture.md).

## When do updates happen?

The Shorebird updater is currently hard-coded to run on app startup. It runs on
a background thread and does not block the UI thread. Any updates will be
installed while the user is using the app and will be applied the next time the
app is restarted.

The Shorebird updater is designed such that when the network is not available,
or the server is down or otherwise unreachable, the app will continue to run
as normal. Should you ever choose to delete an update from our servers, all your
clients will continue to run as normal.

We have not yet added the ability to rollback patches. For now, the simplest
thing is to simply push a new patch that reverts the changes you want to undo.

We expect to add more control over update behavior in the future. Please let us
know if you have needs here, and we're happy to prioritize them.

## Do I need to keep my app_id secret?

No. The `app_id` is included in your app and is safe to be public. You can
check it into version control (even publicly) and not worry about someone
else accessing it.

Someone who has your `app_id` can fetch the latest version of your app from
Shorebird servers, but they cannot push updates to your app or access any
other aspect of your Shorebird account.

## What information is sent to Shorebird servers?

Although Shorebird connects to the network, it does not send any personally
identifiable information. Including Shorebird should not affect your
declarations for the Play Store.

Requests sent from the app to Shorebird servers include:

- app_id (specified `shorebird.yaml`)
- channel (optional in `shorebird.yaml`)
- release_version (versionName from AndroidManifest.xml)
- patch_number (generated as part of `shorebird patch`)
- arch (e.g. 'aarch64', needed to send down the right patch)
- platform (e.g. 'android', needed to send down the right patch)
  That's it. The code for this is in `updater/library/src/network.rs`

## Does Shorebird comply with Play Store guidelines?

Shorebird is designed to be compatible with the Play Store guidelines. However
Shorebird is a tool, and as with any tool, can be abused. Deliberately abusing
Shorebird to violate Play Store guidelines is in violation of the Shorebird
[Terms of Service](https://shorebird.dev/terms) and can result in termination of
your account.

Examples of guidelines you should be aware of, include "Deceptive Behavior" and
"Unwanted Software". Please be clear with your users about what you are
providing with your application and do not violate their expectations with
significant behavioral changes through the use of Shorebird.

Code push services are widely used in the industry (all of the large apps
I'm aware of use them) and there are multiple other code push services
publicly available (e.g. expo.dev & appcenter.ms). This is a well trodden path.

## What platforms does Shorebird support? Does it support iOS?

Current Shorebird is Android-only. We have
[plans to add iOS](https://github.com/shorebirdtech/shorebird/issues/381),
but not yet implemented. Using Shorebird for your Android builds does not affect
your iOS builds. You can successfully ship an appbundle build with `shorebird`
to Google Play and an ipa built with `flutter` to the App Store.
The difference will be that you will be able to update your Android users
sooner than you will your iOS users for now.

We've focused on Android so far on the assumption that most Flutter developers
are targeting Android first. Shorebird can (relatively easily) be made to
support Desktop or embedded targets. If those are important to you, please let
us know.
