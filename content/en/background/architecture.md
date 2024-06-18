{
    "title": "Architecture",
    "weight": 10,
    "description": "TODO"

}

## Introduction

Tweasel provides a toolbox of several tools aimed at analyzing network traffic to detect mobile privacy violations. Here, we try to document how these tools work in a general overview to allow to understand and recreate the result the tools produce.

Generally, the setup the tools were developed for and which we assume here is the following: A computer, which we call the "host", running the tweasel tools, and a mobile device, running either a distribution of Android or iOS, which are connected to each other via USB and are also on the same network. The device might instead also be an emulator running on the host device.

## Programming language & ecosystem

Tweasel is mostly coded using [TypeScript](https://www.typescriptlang.org/), which is a programming language that is transpiled to JavaScript. For transpiling and packaging the project uses [parcel](https://parceljs.org). The scrips can be run on the [nodejs](https://nodejs.org/) runtime. Dependencies are loaded from the [npm package registry](https://www.npmjs.com/).  

Parts of the project, in particular `appstraction` and `cyanoacrylate`, also depend on python scripts. For these, they load and create their own python runtime environment using our TypeScript wrapper library for the python `venv` module, [`autopy`](https://github.com/tweaselORG/autopy). It loads and installs the dependencies from the [Python Package Index (PyPI)](https://pypi.org/).  

`appstraction` and `cyanoacrylate` also require the [Android development tools](https://developer.android.com/tools) to be installed and in the correct version for working with Android devices and emulators. This is automated using the [`andromatic`](https://github.com/tweaselORG/andromatic) helper library. It downloads and installs all the required tools and libraries and manages their usage.

{{< details `How to do the setup manually`>}}
If you want to recreate the results without using our libraries, you'll need to install the dependencies yourself. As well as set up your host for interaction with physical devices, if you want to. For that, you can follow the [documentation of `appstraction`](https://github.com/tweaselORG/appstraction?tab=readme-ov-file#installation).

### Python dependencies

1. [Install python](https://wiki.python.org/moin/BeginnersGuide/Download) (or make sure it is already installed).
2. Create a virtual environments using `python -m venv .venv` (read more in [python’s documentation of venvs](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#create-and-use-virtual-environments)).
3. Install `mitmproxy`, `pymobiledevice3` and `frida-tools` using `pip`: `.venv/bin/pip install mitmproxy pymobiledevice3 frida-tools` (You might need a specific version to have the same environment as we are using. You can check the `scripts/common/python.js` file in the sources of our tools to find the right versions).

### Android development tools

If you want to use Android, you also need to install the [Android development tools](https://developer.android.com/tools), which we recommend to install via [Android Studio](https://developer.android.com/studio). Note that these need to be included in your `PATH`, e.g. by including something like this in your `.zshrc`/`.bashrc`:

```sh
# Android SDK
export ANDROID_HOME="$HOME/Android/Sdk"
export ANDROID_API_VERSION="<Your API version, e.g. 33.0.0>"
export PATH="$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/build-tools/$ANDROID_API_VERSION:$ANDROID_HOME/cmdline-tools/latest/bin/:$ANDROID_HOME/emulator"
```
{{< /details >}}


## Instrumentation of devices and emulators

To ensure a consistent test environment, to place honey data on the device and to automatically control apps and other operating system functions, we employ a number of ways to instrument devices and emulators. Depending on what you want to reproduce, you might or might not need to do these things. Here, we give and overview of the techniques we use to control device functions, for the specific details on how to do them independently, we want to refer you to [`appstraction`’s source code](https://github.com/tweaselORG/appstraction).

### Android

On Android we use two techniques to control the operating system. For one, we make use of the [Android Debug Bridge (`adb`)](https://developer.android.com/tools/adb), which is the official way to interact with Android devices (and emulators). With `adb`, we read out system information, install Apps, change their settings, push files to the device’s file system or simulate user interactions. In particular, we make use of `adb`’s ability to run commands in a shell on the device using `adb shell`.

Because some things we do require elevated privileges, we try to elevate the `adb shell` using the `su` binary: `adb shell su root /bin/sh -c '<command>'`[^1]. To ensure a `su` binary and root privileges are available, we expect the device to either be rooted with an external rooting tool such as [Magisk](https://github.com/topjohnwu/Magisk), or have "Rooted Debugging" enabled, which is an option for USB debugging on some Android distributions. You can find more details on setting up a device for rooted access in [`appstraction`’s README](https://github.com/tweaselORG/appstraction/blob/main/README.md#physical-android-device).

Additionally, we use the instrumentation toolkit [Frida](https://frida.re/), which allows hooking native functions and interacting with the runtime context of a process. This allows us to create scripts that use internal, unexposed APIs to manipulate the device state. We use this for example to access an app’s settings storage and to set the content of the clipboard to place honey data in it. Frida needs root access on the device and the `frida-server` to be installed, which is done automatically in `appstraction`.

{{< details `Manual Frida setup on Android`>}}
You can follow these steps, if you want to manually set up Frida on your device:

1. Check the Frida tools version on your host (see previous section for how to install Frida on your host): `frida --version`
2. [Download `frida-server` in the matching version](https://github.com/frida/frida/releases) (make sure at least the major version match up).  
 You need to choose the `frida-server` binary for the correct architecture of your device (you can determine that using `adb shell getprop ro.product.cpu.abi`).
3. Push the binary to some place on your device using `adb push <path on host>/frida-server <path on device>/frida-server`.
4. Make sure the binary is executable: `adb shell su root /bin/sh -c 'chmod 755 <path on device>/frida-server'`.

Now, if you want to use Frida on the device, you can need to start it using `adb shell su root /bin/sh -c '<path on device>/frida-server --daemonize'`.
{{</ details >}}

### iOS

To control iOS devices, we use [`pymobiledevice3`](https://github.com/doronz88/pymobiledevice3/), a python library that interacts with iOS’s `lockdownd` service and implements internal Apple APIs which are used for communication with the host via USB. Similarly to `adb`, this allows to install apps and retrieve system information, but it does not allow to open a shell on the device.

To do that, the device needs to be jailbroken and an SSH server needs to be installed ([more on which device configuration we require](https://github.com/tweaselORG/appstraction?tab=readme-ov-file#physical-ios-device)). We then connect to the device as root using SSH over the network (the host and the device need to be able to reach each other) and run commands using this shell. We also automatically install all other required dependencies automatically. Using the shell access, we can manipulate app permissions, start apps and sync files from the host to the device.

We also use Frida on iOS, which [can be installed using the Cydia/Sileo](https://frida.re/docs/ios/#with-jailbreak). We use Frida to access several internal APIs, e.g. to set the clipboard content, access the app settings storage, read out device information and start apps.

## Collecting traffic

Device traffic is collected employing a machine-in-the-middle (MITM) attack. In this attack, the devices network traffic is rerouted through the attacking machine (in this case our host machine), saved and potentially manipulated, and then forwarded to the original receiver. This type of attack is (apart from cryptographic checks) opaque to the software running on the device. However, MITM attacks can not access end-to-end-encrypted content. To get around that, the attacker provides its own encryption certificate. If the device is made to trust this certificate, the encrypted traffic can be decrypted and saved by the attacker. Apps can prevent that by requiring a specific certificate, a technique called certificate pinning. To circumvent certificate pinning, additional measures are required.

`cyanoacrylate` uses [`mitmproxy`](https://mitmproxy.org/) to run the MITM attacks on the host. It provides a certificate we need to install on the devices and saves the decrypted traffic, which `cyanoacrylate` then exports in the [HAR file format](https://www.softwareishard.com/blog/har-12-spec/), which contains only the HTTP(S) traffic and ignores other types of network transmissions.

### Android

To reroute the traffic through the host, `cyanoacrylate` sets up a VPN tunnel on the device using [WireGuard](https://www.wireguard.com/). The host then uses `mitmproxy --mode wireguard` to set up a VPN server that can receive the device’s traffic. On the device, we install the [WireGuard app](https://www.wireguard.com/install/#android-play-store-direct-apk-file) and then set up the host as the VPN server. WireGuard allows to either route all the traffic via the VPN or just the network communication generated by a number of whitelisted apps. This allows to attribute requests to the specific app, but it might miss transmissions which are made through a different process, such as Google Cloud Messaging.

In order to read TLS-encrypted traffic, we install the certificate authority generated by `mitmproxy` on the device using `appstraction`. To disable certificate pinning, we use [httptoolkit’s certificate unpinning script](https://github.com/httptoolkit/frida-android-unpinning/blob/4d477da8c58c353a0290ec4829a1de4ca1ca5ae5/frida-script.js), which we load in frida to hook and disable certificate pinning functions in the apps code at runtime. [Our investigation](https://github.com/tweaselORG/meta/issues/16) found that the script is adequate to bypass most certificate pinning methods.

{{< details `How to set up traffic collection on Android`>}}
To manually set up traffic collection on Android, do the following steps:

1. [Download](https://www.wireguard.com/install/#android-play-store-direct-apk-file) and install the WireGuard app on the device.
2. Make sure host and device are on the same network and can reach each other.
3. On the device, open the WireGuard app and tap the plus to add a tunnel. Choose to "Scan from QR code" and scan the code on the host.
4. Activate the tunnel and navigate to <http://mitm.it> on the device. Follow the instructions to install the CA.
5. If necessary, edit the tunnel settings to only tunnel specific apps.
6. Follow [the httptoolkit guide to disable certificate pinning](https://httptoolkit.com/blog/frida-certificate-pinning/).

{{</ details >}}

### iOS

On iOS, VPNs are not easily available to tunnel the device’s traffic. Instead, we use the HTTP proxy settings in the network preferences to route the (HTTP) traffic to the host: The host runs `mitmproxy` in its HTTP proxy mode (the default) and is then set as the proxy server on the device. This sends all of the device’s HTTP traffic to the host and there is no option to filter out specific apps.

To read the TLS-encrypted traffic, the `mitmproxy` CA certificate is installed on the device via the the system preferences. Disabling certificate pinning can be done on jailbroken devices using [the SSLKillSwitch2 Tweak](https://julioverne.github.io/description.html?id=com.julioverne.sslkillswitch2). This is just installed on the device once and then breaks the certificate pinning in (almost) all apps.

{{< details `How to set up traffic collection on iOS`>}}
To manually set up traffic collection on iOs, do the following steps:

0. (If your device is jailbroken: Use Cydia or Sileo to install SSLKillSwitch2. You need to add `https://julioverne.github.io/` to the repositories)
1. Start the proxy on the host: `mitmweb`. A browser should open.
2. Make sure host and device are on the same network and can reach each other.
3. Set up the proxy on the device:
    1. Open the “Settings” app.
    2. Go to “Wi-Fi” and choose the network your device and host are currently connected to.
    3. Scroll down and tap “Configure proxy” and then choose “Manual”.
    4. For “Server” input your devices local IP address, for “Port” input the `mitmproxy` port (default `8080`).
4. Navigate to <http://mitm.it> on the device. Follow the instructions to install the CA.

{{</ details >}}

## Detecting trackers

The recorded traffic in the HAR files is passed to TrackHAR to detect transmissions of data to tracking endpoints. To do so, TrackHAR uses an adapter-based matching approach as opposed to the more common indicator matching, i.e. TrackHAR does not try to recognize specific, pre-known data in the transmission. Instead it has a database of common tracking endpoints and the schema of requests we have seen contacting it. This way, TrackHAR can decode the requests and use the schema to find what data has been transmitted. This makes sure that detected transmissions actually did transmit personal data and did not merely contact a tracking server. [^2]

To create the adapters, we analyze data from test runs of requests we did with thousands of apps and also do other kinds of research to determine what data is transmitted. We [list all ways of our reasoning for determining data types](https://trackers.tweasel.org/research/) and provide [documentation on our research](https://trackers.tweasel.org/) for every adapter in TrackHAR.


[^1]: This command proved to be consistent across different `su` implementations, since Android [uses a non-POSIX-compliant `su`](https://stackoverflow.com/questions/55506205/android-shell-su-doesnt-have-option-c).
[^2]: You can also read [a longer explanation of why we chose this approach](https://trackers.tweasel.org/#what-is-adapter-based-matching).
