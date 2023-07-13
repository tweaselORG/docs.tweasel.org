{
    "title": "Setup",
    "weight": 10,
    "description": "Find out how to install the tweasel libraries and tools and set up your environment for mobile app analysis. Learn how to prepare your devices and emulators on Android and iOS."
}

## Installing the libraries and tools

The [libraries and tools of the tweasel project](/) are distributed as NPM packages. You can install them via NPM or Yarn, if you have [Node.js](https://nodejs.org/) (version 18 or greater) installed. Also, on some systems, you need to have `clang` installed.[^clang-macos] All other dependencies (such as the Android SDK, Frida, and pymobiledevice3) are automatically installed.
You can install Node.js, NPM, and the tweasel CLI on Ubuntu 23.04[^ubuntu-node] like this:

[^clang-macos]: On macOS `clang` should be provided by the Xcode command line tools. You can get it by installing `xcode-select --install`.
[^ubuntu-node]: Note: The repositories of older Ubuntu versions contain versions of Node.js that are too old. On these, you have to install Node.js in another way, for example via [nvm](https://github.com/nvm-sh/nvm) or the [NodeSource packages](https://github.com/nodesource/distributions).

```sh
sudo apt update
sudo apt install nodejs npm clang

sudo npm i -g tweasel-cli
```

The libraries, on the other hand, should be installed locally in a project (i.e. without `-g`), for example for `cyanoacrylate`:

```sh
npm i cyanoacrylate
```

### Setup for physical devices

If you want to work with physical devices, some manual setup is necessary, depending on the platform.

The setup for Android is described in the [Android developer documentation](https://developer.android.com/studio/run/device#setting-up). On Windows, you may need to install the correct [OEM USB driver](https://developer.android.com/studio/run/oem-usb).  
On Ubuntu, the user must be a member of the `plugdev` group (`sudo usermod -aG plugdev <username>`) and `udev` rules for the device must be installed (`sudo apt install android-sdk-platform-tools-common`). Tips for other Linux distributions can be found at [`android-udev-rules`](https://github.com/M0Rf30/android-udev-rules).

For iOS, additional setup steps are only necessary under Windows. Here, you have to install the Apple Device Driver and the Apple Application Support. You can get those by installing iTunes.

## Device preparation

Our libraries work with physical phones and emulators on Android and physical phones on iOS. We test with the following target (but other versions may also work):

| Platform | Type | Tested versions |
| --- | --- | --- |
| Android | `device` (Moto G7 Power) | 13 (API level 33) |
| Android | `emulator` | 11 (API level 30), 13 (API level 33) |
| iOS | `device` (iPhone X, iPhone 6S) | 15.6.1, 15.7.5, 16.0, 16.3.1 |

Depending on what kind of device you want to use for the analysis, the preparation steps described below are necessary. Installing and setting up all other dependencies on the device is done automatically by our tools.

### Physical Android phones

On physical Android phones, USB debugging must be enabled. This can be done via *Settings* -> *System* -> *Developer options* (to activate, tap *Build number* seven times under *Settings* -> *About phone* -> *Android version*) -> *USB debugging*.

To be able to do traffic analysis in a meaningful way, the device needs to be rooted. For this, we recommend [Magisk](https://topjohnwu.github.io/Magisk/). To use that, you also have to unlock the bootloader. The instructions for that vary from device to device. Usually, all data on the device is lost in the process.  
After rooting, root debugging should be enabled if available: *Settings* -> *System* -> *Developer options* -> *Rooted debugging*.
If rooted debugging is not available or not activated, you will be prompted by Magisk to grant `com.android.shell` superuser privileges when `appstraction` tries to use root the first time.

When connecting via USB, you have to confirm that you trust the computer.

### Android emulator

No special setup is required to use an Android emulator. You just have to create an emulator. This can be done through the [Android Studio](https://developer.android.com/studio) Device Manager or via the tweasel CLI (we recommended you configure larger storage space):

```sh
tweasel android-emulator:create "<emulator name>" --partition-size 16384
```

The first run may take a moment as the Android SDK is automatically set up in the background. The command then interactively asks what kind of emulator should be created. For maximum compatibility with apps [we currently recommend](https://github.com/tweaselORG/appstraction/issues/54): Android 11 (API level 30), Android with Google APIs, x86_64.

If you want to set up the emulator further (for example to place honey data), you can start it as follows and then create a snapshot:

```sh
tweasel android-emulator:start "<emulator name>"
tweasel android-emulator:snapshot:create "<snapshot name>" 
```

### Physical iPhones

iOS devices also require a jailbreak. Our tools are tested on iOS 15 and 16 with the [palera1n jailbreak](https://github.com/palera1n/palera1n).[^ios-14] Follow [this guide](https://ios.cfw.guide/installing-palera1n/). Important: The jailbreak must be installed in rootful mode. We strongly recommend to use [palen1x](https://github.com/palera1n/palen1x), which is a Linux distribution for jailbreaking you can boot from a USB Stick, if you are using anything other than macOS.

[^ios-14]: In other projects, we have previously successfully used iOS 14 with the [checkra1n jailbreak](https://checkra.in/). But since we don't have a device with iOS 14 anymore, we can't guarantee that it also works with the tweasel tools.

After the jailbreak, you have to install the `openssh-server` package via Sileo.

When connecting via USB, you have to confirm that you trust the computer.

For optimal use (especially to reduce background traffic of the system and other apps), we also recommend setting the following settings, but this is optional:

* General
    * Background App Refresh: off
    * Software Update
        * Automatic Updates: off
* Display & Brightness
    * Auto-Lock: never
* Privacy & Security
    * Location Services: on
    * Analytics & Improvements
        * Share iPhone Analytics: off
* App Store
    * Automatic Downloads
        * Apps: off
        * App Updates: off
* Accessibility
    * Touch
        * AssistiveTouch: on
