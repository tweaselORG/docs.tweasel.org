{
    "title": "Contributing",
    "weight": 50
}

First of all, thank you very much for your interest in contributing! We warmly welcome contributions and will do our best to help you. On this page, we want to give you an overview of how our project works and how you can contribute.

We invite you to chat with us through Matrix, either through the specific [Tweasel room](https://matrix.to/#/#dade-tweasel:matrix.altpeter.me) or the [space](https://matrix.to/#/#datenanfragen:matrix.altpeter.me) for datarequests.org. Feel free to ask questions, pitch your ideas, or just talk with the community. Alternatively, you can also reach us via [email](mailto:dev@datenanfragen.de).

## Project governance and mission

Tweasel is run and maintained by [Datenanfragen.de e. V.](https://www.datarequests.org/verein/), a non-profit organsation from Germany on a [mission](https://www.datarequests.org/verein/mission-statement/) to advance data protection and privacy in Europe and beyond. In doing so, we stand firmly on the side of the consumer and citizen, whose rights we defend. We manage various projects, including our main site [datarequests.org](https://www.datarequests.org), where we help users exercise their GDPR rights.

With Tweasel, we are tackling the broader ecosystem of online tracking that has become pervasive across the web and mobile apps. Extensive research, including our own, has again and again revealed ubiquitous violations of and a prevalent disregard for data protection law in this context. The vast amounts of personal data collected through such tracking activities are fed into an opaque and shadowy system consisting of thousands of companies, posing very real dangers to users.

Tweasel was [funded by the NLnet Foundation in the NGI0 Entrust Fund](https://nlnet.nl/project/TrackingWeasel/) from January 2023 to June 2024. Decisions on the direction of the project are taken by the board of the non-profit, or ultimately, its members.
## Development

On the [docs homepage]({{< ref "/" >}}), you can find an overview of and introduction to the most important project components and packages that we maintain. Our development happens on [GitHub](https://github.com/orgs/tweaselORG). There you can find a [full list of all repositories](https://github.com/orgs/tweaselORG/repositories?type=all).

Each repository has a README that not only introduces the respective component and how to use it, but will usually also give some instructions on how to get set up for developing on it.

We try to develop in the open. For most features, bug fixes, etc., you will find an associated issue thread, where we introduce our proposal for what to change, sometimes discuss whether and how to implement it, and document our path to implementing the changes. This is inspired by [Simon Willison's concept of issue-driven development](https://simonwillison.net/2022/Nov/26/productivity/). A good example is the issue where we figure out how to [automate certificate authority management on iOS](https://github.com/tweaselORG/appstraction/issues/44). As of the time of writing this, this issue is still open because we still haven't found a way to entirely automate the process. But thanks to what we've documented in the issue, we will be able to quickly catch back after, even after more than a year.  
In the respective pull request, you will also see that each proposed change goes through an extensive review by a team member.

If you are making your own changes, we encourage you to similarly document your steps along the way. Especially don't be afraid to document what went wrong. Generously link to blog posts, StackOverflow threads, etc. that you have found, even if they ultimately didn't end up helping. This will not only make it much easier for us to understand and review your changes or chime in with ideas and suggestions, it will also create a helpful resource for others who are working on adjacent problems.

## What to contribute

As part of our use of issues, we also create a lot of ones for ideas etc. that we would like to implement but don't have the time for in the moment. These are a great place to start if you want to contribute to the project. Have a look at the issues with a [*good first issue*](https://github.com/search?q=org%3AtweaselORG+label%3A%22good+first+issue%22&type=issues) or [*help wanted*](https://github.com/search?q=org%3AtweaselORG+label%3A%22help+wanted%22&type=issues) label. As the name implies, the *good first issue* ones are changes the we believe are a good start to get familiar with the project even if you haven't contributed before. Meanwhile, issues that are only labelled *help wanted* will usually more of your own research and maybe familiarity with the project. Conversely, please be aware that issues without either label are sometimes not a good fit for outside contributors. These may for example be random ideas we've had but haven't discussed yet to decide whether we actually want to implement them.

If you already have a feature that you would like to implement or a bug you would like to fix, that's great, too. For new ideas, please open an issue to discuss them first. We don't want to waste your time and frustrate you if you work on something that isn't a good fit for the project and that we have to reject.

Another good place to start is our [TrackHAR](https://github.com/tweaselORG/TrackHAR) library for detecting tracking data transmissions in traffic recordings. The library mainly relies on custom adapters that we specifically research and write for tracker endpoints. As you can imagine, there's a lot of them out there and so there's always work to be done. Thanks to our [open request database](https://data.tweasel.org/), anyone has public access to hundreds of thousands of real tracking requests that can be used to understand the endpoints. If that's something you're interested in, have a look at our [instructions on how to contribute adapters](https://github.com/tweaselORG/TrackHAR#contributing-adapters).

There are also areas that greatly benefit from non-code contributions. Most importantly, we need help with translations. These are especially needed for our [templates for complaints etc. in ReportHAR](https://github.com/tweaselORG/ReportHAR/tree/main/templates). If you want to help with that, it's best if you reach out via our [i18n room on Matrix](https://matrix.to/#/#dade-i18n:matrix.altpeter.me).

Otherwise, if you are interested in contributing to greater datarequests.org project in other ways besides coding, have a look at our [contribute page](https://www.datarequests.org/contribute) over there for more details.
