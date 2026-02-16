---
layout: post
date: '2015-09-28'
cover: "/images/posts/android-virtual-device-tips-and-tricks/cover"
title: "Android Virtual Device tips and tricks"
summary: "One of the Android Developer's best friends is the emulator (aka AVD). It allows us to simulate some behaviors, like calling, sending SMS messages, simulate a specific location and much more."
categories: [ "Android" ]
tags: [ "lint", "gradle" ]
featured: false
---

One of the Android Developer's best friends is the emulator (aka AVD).
It allows us to simulate some behaviors, like calling, sending SMS messages,
simulate a specific location and much more. In this article, I'm gonna
list some Android Virtual Device tips and tricks

## Creating an emulator

Creating an Android Virtual Device is very simple.

```shell
avdmanager create avd \
    --name myEmulator \
    --device pixel_xl \
    --sdcard 512M \
    --package 'system-images;android-28;google_apis_playstore;x86'
```

To list the devices available run:

```shell
avdmanager list
```

To list the AVD created run:

```shell
emulator -list-avds
```

## Running an emulator:

To run an emulator you can run:

```shell
emulator run -avd myEmulator
```

### Emulator keyboard mapping

In general, I use keyboard mappings for everything. I think it makes me
more productive. AVD has some nice keyboard mappings to simulate the
real experience closer to a real device.

| Command    | Description                   |
|------------|-------------------------------|
| ESC        | Back                          |
| F4         | Hangup/end call button        |
| F7         | Power button                  |
| F8         | Toggle cell networking on/off |
| Ctrl + F5  | Audio volume up button        |
| Ctrl + F11 | Change layout orientation     |

See the complete list on the [Android website](https://developer.android.com/studio/run/emulator-console)

## Logcat

One of the most important tools for the Android developer is
[logcat](http://developer.android.com/tools/help/logcat.html).
Logcat is a simple way to see the system logs and debugging output
from device/emulator. You can use logcat from an
[ADB shell](http://developer.android.com/tools/help/adb.html) to view
the log messages.

```shell
adb logcat
```

## Tips and tricks

The following commands need to be run using telnet

```shell
telnet localhost [emulator port]
```

![App with Tiles](/images/posts/ignore-specific-errors-on-android-lint/android_emulator_port-720w.jpg)

### Receiving a call

```shell
gsm call <telephone-number>
```

### Receiving a SMS

```shell
sms send <telephone-number> <message>
```

### Power battery

```shell
power capacity <battery %>
```

### Network speed

```shell
network speed <download> <upload>
```

To see the list of available options type help inside telnet's prompt.
