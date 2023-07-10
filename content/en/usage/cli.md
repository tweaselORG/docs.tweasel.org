{
    "title": "Traffic analysis with CLI",
    "weight": 20,
    "description": "Learn how to use the tweasel command-line tool to record mobile app traffic and analyse traffic recordings in HAR format for tracking data transmissions."
}

The tweasel CLI allows you to use the features of the different tweasel libraries from the command line without having to write any code yourself.

Its functions are grouped into different commands. The most important ones are:

* `tweasel record-traffic` to record app traffic in HAR format
* `tweasel detect-tracking` to analyse traffic in HAR format for transmitted tracking data
* `tweasel android-emulator` to create, start, and delete Android emulators

All commands have extensive help pages that can be displayed with `--help`, for example:

```sh
tweasel --help
tweasel record-traffic --help
```

In addition, running `tweasel autocomplete` prints instructions on how to set up auto-completion for all commands.

## Recording traffic

To record the traffic of an app, there are two options. If the app is already installed on the phone, you only need to specify the platform:

```sh
tweasel record-traffic --platform android
```

The CLI will then show a list of the installed apps and ask which one you want to record the traffic for. Alternatively, you can also specify the app ID directly, e.g. like this for the Facebook app on Android:

```sh
tweasel record-traffic com.facebook.katana --platform android
```

If the app is not installed yet, you can instead provide its installation file; the platform will be inferred automatically:

```sh
tweasel record-traffic snapchat.apk
```

On iOS, IPA files are supported. On Android, besides single APK files, there is also support for: split APKs, additional OBB files, XAPK files from APKPure, APKM files from APKMirror, and APKS files from SAI.

If you want to start an (already existing) Android emulator, you can do that like this:

```sh
tweasel record-traffic airbnb.apk --run-target emulator --emulator-name "<name of the emulator>"
```

For iOS, two additional arguments have to be provided, the IP address of the phone and of the computer running the tweasel CLI, for example:

```sh
tweasel record-traffic reddit.ipa --ios-ip 10.0.0.3 --ios-proxy-ip 10.0.0.2
```

The traffic will be recorded until you press Enter. If you want to record for a fixed duration instead, you can do that with `--timeout <time in seconds>`. The recorded traffic will be saved as a HAR file in the current directory by default. You can change that with `-o`.  
You also have the option to save different "phases" of the traffic in separate files. You can do that with `--multiple-collections`. Then, after each Enter press, you will be asked if you want to record another phase and, if so, what it should be called. This function is useful, for example, to distinguish the traffic before and after interacting with a consent dialog.

If you want, you can specify `--uninstall-app` to have the app be automatically uninstalled at the end of the analysis.

A few things to note:

* On Android, by default only the traffic of the respective app is recorded (i.e. no background noise from the system and other apps). If you want to record the whole system's traffic instead, you can do that with the `--all-traffic` option.  
  On iOS, however, only the full system traffic can be recorded at the moment.
* By default, all permissions are granted to the apps. You can disable that with `--no-grant-permissions`.
* There are some more options that are not listed here. You can find details about them with `--help`.

## Detecting tracking data transmissions

The `detect-tracking` command helps you identify data transmitted to tracking companies in HAR traffic recordings (regardless of whether they were recorded with our or other tools). To do so, simply pass the HAR file:

```sh
tweasel detect-tracking "<path to HAR file>"
```

The detection is based on adapters for specific tracking endpoints (more on the methodology in the [TrackHAR README](https://github.com/tweaselORG/TrackHAR)). For each request in the HAR file for which we have an adapter, a table of detected transmitted data is displayed. The following information is provided for each transmitted data point:

* `property`: what type of data it is (e.g., `idfa`: phone's advertising ID, `osName`: name of the operating system, `appVersion`: version of the app)
* `context`: the part of the request where this data point was found (e.g. the header or body)
* `path`: where in that context this data point was found
* `value`: the actual transmitted value

If you specify `-x`, we additionally display a reasoning for the assignment (`reasoning`). For each request, there is also a link to the respective adapter in our [tracker wiki](https://trackers.tweasel.org/). There, you can find more technical details about the tracking endpoint and how our decoding works.

{{< hint warning >}}
It is important to keep in mind that the adapter-based method can always only provide a lower bound: The displayed data was definitely transmitted, but it is very possible (likely, even) that more data was transmitted but not detected.
{{< /hint >}}

Therefore, you can also work with honey data and indicators. For this, you pass a list of known attributes of your device to the `detect-tracking` command as a JSON object, such as here:

```sh
tweasel detect-tracking "<path to HAR file>" --indicators '{ "localIp": ["10.0.0.2", "fd31:4159::a2a1"], "idfa": "6a1c1487-a0af-4223-b142-a0f4621d0311" }'
```

In this example, the local IP addresses of the phone and the advertising ID were provided. In the requests that were not covered by any adapter, these are now searched for. We detect not only plain text transmissions, but also base64- and URL-encoded ones. The transmissions detected in this way are also displayed in the table and marked separately.

But here too, it should be noted that data can still be overlooked.

The names for the values (`localIp` and `idfa` in the example) can be chosen freely. Instead of specifying the JSON object directly in the command, you can also create a JSON file with the indicators and pass its path:

```sh
tweasel detect-tracking "<Path to HAR file>" --indicators "<Path to JSON file>"
```
