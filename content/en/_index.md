{
    "title": "Introduction",
    "description": "Tweasel is a project building infrastructure for detecting and complaining about tracking and privacy violations in mobile apps on Android and iOS. This page gives an overview of the tools and libraries which are part of the project."
}

Tweasel is a project building infrastructure for detecting and complaining about tracking and privacy violations in mobile apps on Android and iOS.

As part of this project, we developed a tool suite for automated app analysis and tracking detections. This documentation will explain how the tools work, how you can recreate and verify the results without the tools, and how to use and install them. This page should give you an overview of the tools and their capabilities. You can find the code for all our projects [on GitHub](https://github.com/orgs/tweaselORG/repositories). If you want to follow along with the development, have a look at our [devlog](https://www.datarequests.org/devlog). You can also join our [Matrix room](https://matrix.to/#/#dade-tweasel:matrix.altpeter.me) to get in touch.

The project is run by a privacy-focused non-profit, Datenanfragen.de e.&thinsp;V., who also runs the website [datarequests.org](https://www.datarequests.org), among others. It [receives funds](https://nlnet.nl/project/TrackingWeasel/) from the Zero Entrust fund in the European Commission's Next Generation Internet program via the NLnet foundation.

## Appstraction

[Appstraction](https://github.com/tweaselORG/appstraction) provides an abstraction layer for common instrumentation functions on mobile platforms, specifically Android and iOS. This includes installing, uninstalling, and starting apps, managing their permissions, but also managing devices, like resetting to snapshots, setting the clipboard content, configuring the proxy, etc. Appstraction is built primarily for use in mobile privacy research, but can be used for other purposes as well. It is a library, so you‘ll need to write a script to utilise its features.

## Cyanoacrylate

[Cyanoacrylate](https://github.com/tweaselORG/cyanoacrylate) is the glue attaching `appstraction` to other tools, like `mitmproxy`, to create a simple toolkit run traffic analyses on lots of apps without much user interaction. This is intended especially to analyze the tracking behavior of mobile apps. It supports running apps on Android and iOS, currently on physical devices as well as an emulator for Android.

Among other things, it can:

- Check if device traffic is altered by a DNS blocker
- Start and manage an Android emulator
- Collect HTTP(S) traffic in HAR format
- Automatically manage CA and WireGuard mitmproxy setup

It is also written as a library. For each analysis, you need to write a custom script that fits your specific needs.

## TrackHAR

[TrackHAR](https://github.com/tweaselORG/TrackHAR) is a library for detecting tracking data transmissions from traffic in the HAR format. It utilises two approaches to identify tracking requests:

- Adapter-based parsing, which uses so-called adapters written for specfic types of requests (e.g. requests that go to a specific endpoint). These adapters define algorithms to decode the request, which are often encoded with serveral nested encodings, and match up the decoded content with pre-defined categories of data to identify which data was transmitted. The adapters are documented in [the tracker wiki](https://trackers.tweasel.org).
- Indicator matching, which searches for given strings – indicators – in the requests to identify honey data that had been previously placed on the device. TrackHAR uses this as a fallback if no adapters are available to match a request. Currently, it also supports matching base64- or URL-encoded indicators as well.

## CLI

The [tweasel CLI](https://github.com/tweaselORG/cli) is a commandline utility that combines `TrackHAR`, `cyanoacrylate`, and other utilities to create a single tool for manual mobile tracking analysis. It provides an interface to start and automate traffic recording, manage Android emulators and detect tracking in the recorded requests. If you want to get started with the project, the CLI is a good place to try out the tools.

## parse-tunes/parse-play

[parse-tunes](https://github.com/tweaselORG/parse-tunes) and [parse-play](https://github.com/baltpeter/parse-play) are two libraries to parse data from the Apple App Store and the Google Play Store, respectively. You can use it to get the app ranking charts or access the privacy information app developers provided to the store.

## Open request database

We regularly run large-scale traffic analyses on mobile apps. We are using this data for example to maintain the tracking endpoint adapters of TrackHAR. We want our conclusion there to be replicable by others. In addition, we want to enable as many people as possible to do their own research into trackers.

As such, we are publishing our datasets as open data at [data.tweasel.org](https://data.tweasel.org) for other researchers, activists, journalists, and anyone else who is interested in understanding the inner workings of trackers. The data is published as a Datasette instance, which allows you to interactively explore the full data online, including running arbitrary SQL queries against it.
