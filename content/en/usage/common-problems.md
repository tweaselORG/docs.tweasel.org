{
    "title": "Common problems",
    "weight": 30,
    "description": "Discover some common problems that you may encounter when using our tools and libraries and how to solve them. Get tips and tricks for dealing with issues related to installation, device compatibility, traffic recording, and tracking detection."
}

Here we describe some common problems that you may encounter when using our tools and libraries and how to solve them. If you have a problem that is not described here, please feel free to open an issue in the respective repository on GitHub or to reach out through our [Matrix room](https://matrix.to/#/#dade-tweasel:matrix.altpeter.me).

## `Failed to install app: App doesn't support device's architectures` in `appstraction`

When installing Android apps with `appstraction`, `cyanoacrylate`, or `tweasel-cli`, you may see an error like this:

```
Error: Failed to install app: App doesn't support device's architectures (x86_64,arm64-v8a).
    at Object.installMultiApk (~/appstraction/src/android.ts:306:23)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at Object.installApp (~/appstraction/src/android.ts:544:21)
    at <anonymous> (~/appstraction/examples/android-emulator.ts:25:5)
```

If you are installing the app manually through `adb`, you would instead see: `INSTALL_FAILED_NO_MATCHING_ABIS: INSTALL_FAILED_NO_MATCHING_ABIS: Failed to extract native libraries, res=-113`

While the Java code of apps should work regardless of the device's CPU architecture, Android apps can also contain native code that is compiled for specific CPU architectures only. You will see this error if you try to install an app that contains native code that is only compiled for CPU architectures that your device does not support.

To find out the architectures your device supports, you can run:

```sh
adb shell getprop | grep ro.product.cpu.abi
```

You may be able to obtain a different version of the app that supports your phone's architecture. How this works depends on your method for downloading APKs. For example, the [`googleplay` CLI tool](https://github.com/1268/google-play) has a `-p` option to set the platform and sites like APKPure and APKMirror will display the supported architectures for each downloadable APK version.

However, it is also possible that the app does not support your phone's architecture at all and no compatible version exists. In this case, you will unfortunately have to use a different device or emulator to run the app.

{{< hint info >}}
**Note**: If you are running apps in an Android emulator, we recommend using Android 11 x86_64 emulators for the best compatibility. This particular combination supports apps compiled for all relevant architectures: x86_64, x86, arm64-v8a, armeabi-v7a, armeabi. Notably, it can also [run ARM apps with good performance](https://android-developers.googleblog.com/2020/03/run-arm-apps-on-android-emulator.html).  
This unfortunately [does not appear to be the case for newer Android versions](https://github.com/tweaselORG/appstraction/issues/54).
{{< /hint >}}

## Only one app's traffic is recorded by `cyanoacrylate`

When recording network traffic with `cyanoacrylate` or `tweasel-cli`, you may notice that the HAR file only contains traffic from the app you ran an analysis on but not from other apps or the system. This is expected. By default, both `cyanoacrylate` and `tweasel-cli` only record the traffic of the specified app to facilitate attribution. But you can change this behaviour.

In `cyanoacrylate`, if you run `startTrafficCollection()` on an `AppAnalysis`, you will only ever get this app's traffic. However, you can also start a traffic collection on an `Analysis` and precisely specify which apps' traffic you want to record:

```ts
// Collect traffic from all apps, including the system.
analysis.startTrafficCollection({ mode: 'all-apps' });
// Collect traffic only from the specified apps.
analysis.startTrafficCollection({
    mode: 'allowlist',
    apps: ['com.example.app1', 'com.example.app2'],
});
// Collect traffic from all apps except the specified apps.
analysis.startTrafficCollection({
    mode: 'denylist',
    apps: ['com.example.app1', 'com.example.app2'],
});
```

In `tweasel-cli`, you can use the `--all-traffic` option to record all traffic:

```sh
tweasel record-traffic app.apk --all-traffic
```

{{< hint warning >}}
**Note**: App filtering is currently only supported on Android. On iOS, we always record all traffic.
{{< /hint >}}

## Recording is missing traffic due to problems with certificate pinning bypass

Some apps use a technique called certificate pinning to verify the identity of the servers they communicate with. This means that they only accept connections from servers that present a specific certificate that the app knows in advance. This is a security measure to prevent machine-in-the-middle attacks, but it also prevents us from recording the app's traffic by using a proxy with our own certificate.

To record the traffic of such apps, we need to bypass the certificate pinning mechanism. This can be done by using a tool called [Frida](https://frida.re/), which allows us to inject code into the app's process and modify its behavior at runtime. We use [frida-android-unpinning](https://github.com/httptoolkit/frida-android-unpinning), a script that tries to disable certificate pinning for various common libraries and frameworks.

However, this script may not work for all apps, as some may use custom or uncommon pinning methods that are not covered by the script. You can recognize certificate pinning related errors by looking at the `mitmproxyEvents` in the analysis results. If it contains `tlsFailed` events with error messages like `The client does not trust the proxy's certificate for`, these are likely related to certificate pinning. If you encounter problems with the unpinning script, it's best to report them [upstream](https://github.com/httptoolkit/frida-android-unpinning/issues). If you think the problem is significant and relevant to many users, you can also report it to us and we might look into it if we have the capacity.

If you are interested in writing your own hook to bypass certificate pinning for a specific app, [this guide](https://httptoolkit.com/blog/android-reverse-engineering/) is a great resource to learn how to use Frida and reverse engineer Android apps.

## TrackHAR doesn't recognize all tracking traffic in a HAR file

When using TrackHAR or `tweasel-cli` to detect tracking data transmissions in a traffic recording in HAR format, you may notice that there are transmissions that are missed. This is expected. TrackHAR can only ever provide a lower bound on the transmitted tracking data: The reported data was definitely transmitted, but it is very possible (likely, even) that more data was transmitted but not detected.

In fact, when in doubt, we deliberately choose to avoid overmatching and false positives. We want to ensure that the data we report is accurate and reliable, and not based on guesses. This is especially important for research and legal enforcement purposes.

By default, we use adapters to parse specific tracking endpoints. These are custom functions we wrote that know how to decode and extract the transmitted data from the request content. We have written adapters for the most common and popular tracking endpoints, but there are many more that we have not covered yet. Adapters also need to be updated regularly as tracking companies may change their formats or obfuscation methods.

You can also enable indicator matching as a fallback for requests not handled by any adapter. Indicator matching relies on you providing known honey data values (such as the advertising ID or geolocation) that are then searched for in the requests. Here's an example of how to specify indicator values in TrackHAR:

```ts
const indicators = {
    localIp: [ '10.0.0.2', 'fd31:4159::a2a1' ],
    idfa: '6a1c1487-a0af-4223-b142-a0f4621d0311'
};

const data = await processHar(JSON.parse(har), { indicatorValues: indicators });
```

In the tweasel CLI, you can specify indicator values with `--indicators`, either inline or as a JSON file:

```sh
# Use indicators provided inline.
tweasel detect-tracking app.har --indicators '{ "localIp": ["10.0.0.2", "fd31:4159::a2a1"], "idfa": "6a1c1487-a0af-4223-b142-a0f4621d0311" }'
# Use indicators provided in a JSON file.
tweasel detect-tracking app.har --indicators indicators.json
```

But indicator matching also has limitations. It can only detect static values that you provide, not dynamic or low-entropy values such as free disk space or operating system version. It can also miss values that are encoded or obfuscated in ways that TrackHAR does not support yet.

If you find a request that you think should be detected but isn't, please [open an issue on GitHub](https://github.com/tweaselORG/TrackHAR/issues).

## Installation of `appstraction`, `cyanoacrylate`, or `tweasel-cli` takes a long time

When you install `appstraction`, `cyanoacrylate`, or `tweasel-cli`, we automatically download and install a lot of necessary dependencies for you (this includes for example the Android SDK, `mitmproxy`, Frida, and `pymobiledevice3`). This can take a while. How long it takes exactly depends heavily on your computer and internet connection, so it is hard to give an estimate of what is a normal duration.

However, if the installation takes longer than 30 minutes, something is probably wrong and you have likely encountered a bug. Please open an issue in the respective GitHub repository or reach out through our [Matrix room](https://matrix.to/#/#dade-tweasel:matrix.altpeter.me).

## Physical device doesn't work with `appstraction`, `cyanoacrylate`, or `tweasel-cli`

If you are unsuccessfully trying to use a physical Android device or iPhone with `appstraction`, `cyanoacrylate`, or `tweasel-cli`, make sure that you have followed the [setup instructions](/usage/setup#) carefully. While we have automated most of the preparation steps, some things still need to be done manually. Here are a few things that might have gone wrong.

If you are using an Android device:

* Depending on your operating system, you may need to install an OEM USB driver or correctly setup permissions and `udev` rules. Refer to the [Android developer documentation](https://developer.android.com/studio/run/device#setting-up) for details.
* You need to make sure to confirm that you trust the computer when connecting via USB.
* If you are using features that require a rooted phone, you need to either enable *Rooted debugging* in the *Developer options* (this works on LineageOS, but as far as we can tell, this is generally not an option on stock Android), or grant `com.android.shell` superuser privileges in Magisk (you should see a prompt for that when you run your first analysis).

If you are using an iPhone:

* On Windows, you need to install the Apple Device Driver and the Apple Application Support. You can get those by installing iTunes.
* You need to make sure to confirm that you trust the computer when connecting via USB.
* For the jailbreak using palera1n, you need to use rootful mode (install the jailbreak with `palera1n -fc` to enable rootful mode and subsequently start the iPhone with `palera1n -f`).
* Make sure that you have installed the `openssh-server` package via Sileo.
* If you have not used the default password (`alpine`) in the palera1n setup wizard, you need to provide the password you used.

## `Cannot find AVD system path. Please define ANDROID_SDK_ROOT` when running existing emulator with `andromatic`

When running an Android emulator with `andromatic` or `tweasel-cli` that you have previously created manually, you may see an error like this:

```
INFO    | Android emulator version 32.1.13.0 (build_id 10086546) (CL:N/A)
INFO    | AVD andromatic-test has path ~/.android/avd/andromatic-test.avd
INFO    | trying to check whether ~/Library/Caches/andromatic is a valid sdk root
WARNING | ~/Library/Caches/andromatic/system-images/android-33/google_apis/x86_64/ is not a valid directory.
WARNING | emulator has searched the above paths but found no valid sdk root directory.
PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT
```

This can happen if you have not installed the corresponding the system image with `andromatic` yet. To fix this, you can run:

```sh
andromatic-install -p <package name>
```

You can infer the correct package name from the error message. In this example, the command would be:

```sh
andromatic-install -p "system-images;android-33;google_apis;x86_64"
```

{{< hint info >}}
If you create emulators with `andromatic` or the tweasel CLI instead, you don't have to worry about this. `andromatic` will automatically install the necessary system image for you.
{{< /hint >}}

**Background**: The Android SDK uses [two different main folders](https://developer.android.com/tools/variables#envar): `ANDROID_HOME` (which contains the SDK components) and `ANDROID_USER_HOME` (which contains user-specific preferences and data). Notably, [emulator AVDs are stored under `ANDROID_USER_HOME`](https://developer.android.com/tools/variables#android_emulator_home).

`andromatic` automatically creates and manages [its own `ANDROID_HOME` directory](https://github.com/tweaselORG/andromatic/blob/main/README.md#andromatic) that is separate from any other `ANDROID_HOME` you may have manually installed yourself. It however does not touch your `ANDROID_USER_HOME` at all. This means that you can use `andromatic` to interact with emulators that you may have created manually using your own Android SDK installation. *However*, it can happen that you have installed the necessary system images in your own `ANDROID_HOME` but not in `andromatic`'s `ANDROID_HOME`.

## `SyntaxError: Unexpected token '?'` when installing `appstraction`

When installing `appstraction`, `cyanoacrylate`, or `tweasel-cli`, you may see an error like this:

```
npm ERR! code 1
npm ERR! path ~/appstraction-test/node_modules/appstraction
npm ERR! command failed
npm ERR! command sh -c node scripts/postinstall.js; andromatic-install -p platform-tools 'build-tools;33.0.2'
npm ERR! file://~/appstraction-test/node_modules/autopy/dist/index.js:31
npm ERR!     const version = versionRange ?? ">= 0";
npm ERR!                                   ^
npm ERR! 
npm ERR! SyntaxError: Unexpected token '?'
npm ERR!     at Loader.moduleStrategy (internal/modules/esm/translators.js:133:18)
npm ERR! file://~/appstraction-test/node_modules/andromatic/dist/cli-install.js:33
npm ERR!     ].filter((v)=>options?.allowPrerelease === undefined || options?.allowPrerelease || !v.includes("-rc")).sort((a, b)=>{
npm ERR!                           ^
npm ERR! 
npm ERR! SyntaxError: Unexpected token '.'
npm ERR!     at Loader.moduleStrategy (internal/modules/esm/translators.js:133:18)
npm ERR!     at async link (internal/modules/esm/module_job.js:42:21)
```

This can happen if you are using a Node.js version that is too old. Our tools and libraries require Node.js 18 or newer.

{{< hint warning >}}
Unfortunately, some Linux distributions (e.g. Ubuntu 22.04 and older) ship very old versions of Node.js in their package repositories. On these, you have to install Node.js in another way, for example via [nvm](https://github.com/nvm-sh/nvm), the [NodeSource packages](https://github.com/nodesource/distributions), or by [downloading the binaries directly](https://nodejs.org/en/download).
{{< /hint >}}

## `libcrypt.so.1: cannot open shared object file` in `autopy`

When installing `appstraction`, `cyanoacrylate`, or `tweasel-cli` or when using `autopy`, you may see an error like this:

```
Error: Command failed with exit code 127: ~/.cache/autopy/python/3.11.3/bin/python3 -m venv ~/.cache/autopy/venv/cyanoacrylate
~/.cache/autopy/python/3.11.3/bin/python3: error while loading shared libraries: libcrypt.so.1: cannot open shared object file: No such file or directory
```

This can happen if you are using a Linux distribution that does not ship with `libcrypt.so.1` by default. To fix this, you can install the `libxcrypt-compat` package. For example, on Arch Linux, you can run `sudo pacman -S libxcrypt-compat`.

**Background**: `autopy` uses [`python-build-standalone`](https://gregoryszorc.com/docs/python-build-standalone/main/index.html) to download a portable Python distribution that works regardless of which Python version you may (or may not) already have installed on your system. [`python-build-standalone` expects `libcrypt.so.1` to be present](https://gregoryszorc.com/docs/python-build-standalone/main/quirks.html#missing-libcrypt-so-1) as mandated by the Linux Standard Base Core Specification, which most distributions do in fact conform to.

## `appstraction` hangs indefinitely in `ensureDevice()` after phone crashes

In large-scale analyses with `appstraction` or `cyanoacrylate`, we have sometimes [observed](https://github.com/tweaselORG/appstraction/issues/102) the phone crashing and rebooting with the following error being shown:

```
~/appstraction-test/node_modules/execa/lib/error.js:59
		error = new Error(message);
		        ^


Error: Command failed with exit code 224: /home/<user>/.cache/andromatic/platform-tools/adb shell am broadcast -a com.wireguard.android.action.SET_TUNNEL_UP -n 'com.wireguard.android/.model.TunnelManager$IntentReceiver' -e tunnel appstraction
Broadcasting: Intent { act=com.wireguard.android.action.SET_TUNNEL_UP flg=0x400000 cmp=com.wireguard.android/.model.TunnelManager$IntentReceiver (has extras) }
cmd: Failure calling service activity: Broken pipe (32)
    at makeError (~/appstraction-test/node_modules/execa/lib/error.js:59:11)
    at handlePromise (~/appstraction-test/node_modules/execa/index.js:124:26)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at Object.setProxy (~/appstraction-test/node_modules/appstraction/dist/index.js:1026:17) {
  shortMessage: "Command failed with exit code 224: /home/<user>/.cache/andromatic/platform-tools/adb shell am broadcast -a com.wireguard.android.action.SET_TUNNEL_UP -n 'com.wireguard.android/.model.TunnelManager$IntentReceiver' -e tunnel appstraction",
  command: "/home/<user>/.cache/andromatic/platform-tools/adb shell am broadcast -a com.wireguard.android.action.SET_TUNNEL_UP -n 'com.wireguard.android/.model.TunnelManager$IntentReceiver' -e tunnel appstraction",
  escapedCommand: `"/home/<user>/.cache/andromatic/platform-tools/adb" shell am broadcast -a com.wireguard.android.action.SET_TUNNEL_UP -n "'com.wireguard.android/.model.TunnelManager$IntentReceiver'" -e tunnel appstraction`,
  exitCode: 224,
  signal: undefined,
  signalDescription: undefined,
  stdout: 'Broadcasting: Intent { act=com.wireguard.android.action.SET_TUNNEL_UP flg=0x400000 cmp=com.wireguard.android/.model.TunnelManager$IntentReceiver (has extras) }\n' +
    'cmd: Failure calling service activity: Broken pipe (32)',
  stderr: '',
  failed: true,
  timedOut: false,
  isCanceled: false,
  killed: false
}
```

Afterwards, starting a new analysis doesn't work because `appstraction` hangs indefinitely in `ensureDevice()`. The crash and hanging seem to be caused by Frida. To verify this, you can run:

```sh
adb shell /data/local/tmp/frida-server --version
```

If this hangs indefinitely, too, then you have the same problem. To fix the problem, you just need to kill the stuck Frida process:

```sh
adb shell pkill -9 frida-server
```

Alternatively, you can also manually reboot the phone.

## `Emulator failed to start: Command was killed with SIGSEGV (Segmentation fault)` in headless mode (`-no-window`) in `cyanoacrylate`

This might be caused by a bug in the Android emulator’s hardware acceleration. [In the case we encountered](https://github.com/tweaselORG/cyanoacrylate/issues/53), the emulator still starts without problems if `headless` is set to `false`. You can try to resolve the problem by disabling hardware acceleration (see also [the `-accel` option of the Android emulator](https://developer.android.com/studio/run/emulator-commandline#common)):

```js
{
    startEmulatorOptions: {
        emulatorName: '<your-emulator>',
        headless: true,
        hardwareAcceleration: {
            mode: 'off',
        },
    },
}
```

Alternatively, if you want to keep hardware acceleration, you can try different options for the GPU acceleration to find what works for you (take a look at [the `-gpu` option of the Android emulator](https://developer.android.com/studio/run/emulator-acceleration#accel-graphics)):

```js
{
    startEmulatorOptions: {
        emulatorName: '<your-emulator>',
        headless: true,
        hardwareAcceleration: {
            gpuMode: 'host',
        },
    },
}
```
