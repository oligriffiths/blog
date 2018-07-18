title: Ember Cli & Broccoli 2.0
tags:
- Ember
- js
date: 2018-07-17 12:00:00
---

I recently completed work on updating [Ember-CLI](https://embercli.com) to use the new [Broccoli 2.0](https://github.com/broccolijs/broccoli)
release. This is currently behind an experiment flag. Read on to find out how you can test it out.

![broccoli](/broccolijs/assets/broccoli-logo.png)

<!-- more -->

## History

First, a bit of history. Ember-CLI forked Broccoli back in 2015 to [broccoli-builder](https://github.com/ember-cli/broccoli-builder)
in order to add features to the core library that were not accepted as upstream changes. From that point on, work happened
in parallel, with updates being back-ported from the Broccoli master repo into the fork. Since then multiple contributions
to the Broccoli master repo have been accepted, and in 2017, Stefan Penner [@stefanpenner](https://twitter.com/stefanpenner)
and Robert Jackson [@rwjblue](https://twitter.com/rwjblue) did a huge amount of work rewriting Broccoli into ES6 format,
and re-architected the internals.

As such, the divergence has become so great that backporting is no longer possible. That, along with some issues with
`broccoli-builder` that folks have had, especially with the `tmp` directory being within the project directory, meant that
the Ember-CLI team decided to back to upstream some fixes from `broccoli-builder` into `master`, and then update Ember-CLI
to use the newly updated Broccoli master.

I was fortunate enough to be invited to join the Ember-CLI face-to-face after [EmberConf 2018](http://emberconf.com/)
in Portland earlier this year, due to my work on the Broccoli workshop that I presented at the conference and a
[tutorial series](/broccolijs) that I produced. During this meeting, we formulated a plan of attack, and I drew up a
quest issue ([issue 360](https://github.com/broccolijs/broccoli/issues/360)).

Work then began over the following months, to upstream some required changes that had been made to `broccoli-builder` into
the main Broccoli repository, as well as integrating the main Broccoli library back into Ember-CLI in parallel with
`broccoli-builder`, so that it can be tested side-by-side under the experiment functionality of Ember-CLI.

## Usage

In order to test out the new Broccoli 2.0 release in Ember-CLI you must do 2 things, update Ember-CLI to use `master`
and to enable the experiment. Experiments are only enabled in Ember-CLI master, so first, let's get that:

### Ember-CLI master

```
yarn add --dev "https://github.com/ember-cli/ember-cli.git#master"
```
or
```
npm install --save-dev "https://github.com/ember-cli/ember-cli.git#master"
```

Once that completes, you'll be using the bleeding-edge Ember-CLI, exciting!

### Enabling Broccoli 2.0

Next, you need to enable a build using the experiment, to do that, run:

```
EMBER_CLI_BROCCOLI_2=true ember build
```

This should build as before, hopefully everything works and you don't get any errors.

Now try running serve:

```
EMBER_CLI_BROCCOLI_2=true ember serve
```

If you experience any errors when using the experiment enabled that you do not get when running without the experiment,
please hop onto the [Ember community slack](https://embercommunity.slack.com/messages/C0CRP360G) and let us know, or
submit an issue to [github](https://github.com/ember-cli/ember-cli/issues), please make sure you include:

* what OS you're running
* what Ember version you're using
* what addons and broccoli-plugins you're using
* a link to your project if it's publicly available.

### Enabling system temp directory

One problem with `broccoli-builder` is that it uses the project directory to write temp files during the build process.
This often causes performance problems, especially on windows. Broccoli master now supports (by default) setting the temp
directory to the system temp directory. To enable this in Ember-CLI, do the following:

Start by removing the temp directory if it already exists:
```
rm -rf tmp;
```
Now build:
```
EMBER_CLI_BROCCOLI_2=true EMBER_CLI_SYSTEM_TEMP=true ember build
```
Or to serve:
```
EMBER_CLI_BROCCOLI_2=true EMBER_CLI_SYSTEM_TEMP=true ember serve
```

When serving, you should notice that the `tmp` directory is not created in the project. Note that `EMBER_CLI_SYSTEM_TEMP` only
works in combination with `EMBER_CLI_BROCCOLI_2`.
Please test this out and report any issues, or performance improvements/degradations as above.

I hope you enjoy using the new Broccoli 2.0. Next up, documentation....
