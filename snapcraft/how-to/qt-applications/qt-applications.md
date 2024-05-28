## Why are snaps good for Qt5 and KDE Frameworks applications?

Snapcraft bundles necessary libraries required by the application, and can configure the environment for confinement of applications for end user peace of mind. Developers can ensure their application is delivered pre-packaged with libraries which will not be replaced or superseded by a distribution vendor.

Here are some snap advantages that will benefit many Qt 5 applications:

* **Snaps are easy to discover and install**\
  Millions of users can browse and install snaps graphically in the Ubuntu Software Center, the Snap Store or from the command-line.
* **Snaps install and run the same across Linux**\
  They bundle the exact version of whatever is required, along with all of your app's dependencies, be they binaries or system libraries.
* **You control the release schedule**\
  You decide when a new version of your application is released without having to wait for distributions to catch up.
* **Snaps automatically update to the latest version**\
  Four times a day, users' systems will check for new versions and upgrade in the background.
* **Upgrades are safe**\
  If your app fails to upgrade, users automatically roll back to the previous revision.

### Build a snap in 20 minutes

In this how-to, we will create a snap of KDE's calculator application [KCalc](https://apps.kde.org/kcalc/). This application uses Qt 5 and the KDE Frameworks.

> ⓘ  For a brief overview of the snap creation process, including how to install `snapcraft` and the role of *snapcraft.yaml* in defining a snap, see [Snapcraft overview](/t/snapcraft-overview/8940). For a more comprehensive breakdown of the steps involved, take a look at [Creating a snap](/t/creating-a-snap/6799).

We'll start by preparing a *snapcraft.yaml* file that builds a *mostly* working development version of KCalc. We'll then look at a few ways in which we can improve our *snapcraft.yaml* and submit our snap to the Snap Store.

Typically this guide will take around 20 minutes to follow. Once complete, you'll understand how to package Qt 5 applications as snaps and deliver them to millions of Linux users. After making the snap available in the store, you'll get access to installation metrics and tools to directly manage the delivery of updates to Linux users.

## Getting started

The full version of our *snapcraft.yaml* file can be found below. We'll explain each line of this file in the following sections.

[details=Click here to expand the initial *snapcraft.yaml* for KCalc]
```yaml
name: kcalc-example
adopt-info: kcalc
grade: stable

base: core22
confinement: devmode

apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions: [ kde-neon ]

parts:
  kcalc:
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    parse-info: [ usr/share/metainfo/org.kde.kcalc.appdata.xml ]
    plugin: cmake
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"
```
[/details]

#### Top-level metadata

The *snapcraft.yaml* file starts with a small amount of human-readable metadata. This data is used in the presentation of your app in the Snap Store. See [Snapcraft top-level metadata](/t/snapcraft-top-level-metadata/8334) for more information.

```yaml
name: kcalc-example
adopt-info: kcalc
grade: devel
```

Every snap needs a `name`. Valid snap names consist of lower-case alphanumeric characters and hyphens. Names must contain at least one letter and they also cannot start or end with a hyphen. Names also cannot be more than 40 characters long. If you want to publish your snap, you'll need specify a `name` that hasn't already been taken in the Snap Store.

The `adopt-info` attribute tells Snapcraft that the mandatory `summary`, `description` and `version` attributes will be set during the `kcalc` build part, later on in *snapcraft.yaml*. We're doing this as Snapcraft is able to extract this information - as well as the optional `title` and `icon` - from an [AppStream metadata file](/t/using-external-metadata/4642) included with the KCalc source code.

> ⓘ If you want to snap an application that doesn't include an external metadata file, then you should delete the `adopt-info` line and add separate entries for the `summary`, `description`, `version`, `title` and `icon` attributes instead.

The `grade` is optional. It defines the *quality* of the snap. We're specifying the `grade` as `devel` to tell Snapcraft that this snap is in development, and that it isn't suitable to be published to the `stable` or `candidate` release [channels](/t/channels/551) of the Snap Store.

#### Base

The `base` keyword defines a special kind of snap that provides a run-time environment with a minimal set of libraries that are common to most applications. Base snaps are transparent to users, but they need to be considered, and specified, when building a snap. See [Base snaps](/t/base-snaps/11198) for more information.

```yaml
base: core22
```

We're setting the base to [`core22`](https://snapcraft.io/core22) as it is the only base currently supported by the `kde-neon` extension that we rely on later on.

#### Security model

The `confinement` defines how your snap is [isolated](/t/snap-confinement/6233) from the user's system.

```yaml
confinement: devmode
```

It's generally worth setting the `confinement` to `devmode` when you start working on a new snap. `devmode` enables you to check that the snapped application works *without* the security restrictions imposed by the snap daemon. It also helps you to identify the [interfaces](/t/supported-interfaces/7744) that your snap needs. See [Debug snaps](/t/debugging-snaps/18420) for more information.

Once your snap is ready, you'll need to re-build your snap with `strict` level `confinement` and test that it still works, before you can make it publicly available to users of the Snap Store. (`devmode` snaps may only be released to the hidden "edge" channel of the Snap Store where you and other developers can install them.)

#### Apps

Apps are the commands and services exposed to end users. We define just one app, which we name `kcalc-example` - but it's worth noting that snaps can contain multiple apps.

As we use `kcalc-example` as both the name of our *app* and the name of our *snap*, users will be able to launch the app by running `kcalc-example` in a terminal.

However, if the names differ, then the command to run the app would instead take the form `<app name>.<snap name>`. (So, if we kept our snap's name as `kcalc-example`, but changed the app name to `calculator`, then the command to run our app would be `kcalc-example.calculator`.) Using the snap name as a prefix ensures that we avoid conflicts with apps defined by other installed snaps. If you don't want the command to include a prefix, consider applying for an alias on the [Snapcraft forum](https://forum.snapcraft.io/t/455). Any aliases approved by the Snap Store's review team will be set up automatically when a user installs your snap from the Snap Store.

```yaml
apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions: [ kde-neon ]
```

The `common-id` attribute links our app to an AppStream *component*, from which Snapcraft is able to extract and the details of the relevant [desktop entry file](/t/desktop-menu-support/13115). This saves us from having to specify the desktop entry file manually using a `desktop` attribute. See [Using AppStream metadata](/t/4642#heading--appstream) for more information.

The `command` is the path to the app's main executable file. 

The [`kde-neon`](/t/the-kde-neon-extension/13752) extension ensures that the relevant Qt/KDE library snaps are available at build time and run time. The extension also:
- adds `plugs` to the `desktop`, `desktop-legacy`, `opengl`, `wayland` and `x11` interfaces
- configures the run time environment of the app, so that all desktop functionality is correctly initialised

#### Parts

Parts define how to build your app. We define just one in our example, which we name `kcalc`.

```yaml
parts:
  kcalc:
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    parse-info: [ usr/share/metainfo/org.kde.kcalc.appdata.xml ]
    plugin: cmake
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"
```

`source` tells Snapcraft where to find the source code for this part. Whilst we're linking to a compressed tarball hosted on a public website, Snapcraft is also able to retrieve source code from a version control system or a local directory. 

> ⓘ We're specifically using version 23.08.5 of KCalc released in February 2024 for this how-to. At the time of writing (May 2024) this is the latest release that supports Qt 5.

We set `parse-info` to the location of the AppStream metadata file in our snap. This file will be used by the `adopt-info` and `common-id` attributes discussed above.

[The `cmake` `plugin`](/t/the-cmake-plugin/8621) tells Snapcraft that the KCalc source code should be built using the *CMake build system*.
 
In addition to Qt 5 and the KDE Frameworks, KCalc depends on two libraries: MPFR and GMP. We use:
- `build-packages` to ensure that the *development* versions of these libraries are during the build process, and
- `stage-packages` to bundle the *run time* versions within the snap.
The packages will be obtained from the Ubuntu *apt* repositories applicable to your chosen `base` (so, as we're using `core22`, the packages will need to be available in *Ubuntu 22.04*). We don't need to specify the Qt or KDE libraries here, as the `kde-neon` extension handles this for us.

We can pass various parameters to CMake using `cmake-parameters`. These are mostly self-explanatory (i.e. building in 'release' mode, defining the locations for the build and skipping build tests) however it's worth briefly discussing the final two:
- `"-DINSTALL_ICONS=ON"` embeds the KCalc icons within the snap, so that it correctly appears in the user's application launcher (and the Snap Store). If this parameter is omitted, then you may see a default icon instead.
- `"-DKF5DocTools_FOUND=OFF"` disables the generation of documentation files for many KDE applications. This slightly reduces the size of our snap, and users will still be able to access the documentation online via KCalc's *Help* menu.

### Building the snap

You can download our example repository by running the following command:

```bash
git clone https://github.com/snapcraft-docs/kcalc-example
```

The *snapcraft.yaml* file that we've discussed so far can be found in the *initial* directory.

You can build the snap by simply executing the *snapcraft* command in that directory. The build should take about five to ten minutes to complete. You'll see *snapcraft* display various status messages in your terminal whilst the build is ongoing. If all goes well, once the command has finished you should see something like the following:

```bash
$ snapcraft
Generated snap metadata
Created snap package kcalc-example_23.08.5_amd64.snap
```

> ⓘ We have only tested this example on an *amd64* system, but the `kde-neon` extension is also available for *arm64*. If you're able to build the snap on an *arm64* system, please let us know in our [Discourse thread](https://forum.snapcraft.com/t/qt5-and-kde-frameworks-applications/13753).

The resulting snap can be installed locally. This requires the `--dangerous` flag because the snap is not signed by the Snap Store.

```bash
$ sudo snap install kcalc-example_23.08.5_*.snap --dangerous
```

You can then try it out:

```bash
$ snap run kcalc-example
```

Or, alternatively:

```bash
$ kcalc-example
```

Removing the snap is simple too:

```bash
$ sudo snap remove kcalc-example
```

If needed, you can clean up the build environment with the following command:

```bash
$ snapcraft clean
```

By default, when you make a change to *snapcraft.yaml*, Snapcraft will only build the parts that have changed. Cleaning a build forces your snap to be rebuilt in a clean environment and will take longer.

### Improving our snap

You might notice a few peculiarities when testing our snap, for example:
- sounds might not play properly, or at all 
- the application might use a different set of cursors to the rest of your applications

Let's improve the user experience by addressing them.

#### Audio support

KCalc will attempt to play a sound when an error occurs, for example if user tries to input multiple decimal places for a given number. However, our version of KCalc isn't able to play these sounds.

To start with, we need to update *snapcraft.yaml* to grant our snap access to the `audio-playback` interface. To do this, we can add a `plugs` entry with the value `audio-playback` to the `kcalc-example` app definition.

The result looks like this:

```yaml
apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions:
      - kde-neon
    plugs:
      - audio-playback
```

KCalc also relies upon the audio library *libcanberra-pulse* to play sounds.

> KCalc doesn't link to this library correctly, so we wouldn't have received any errors when building our snap to say that it was missing. Instead, we used the debugging tool *strace* to find out what library KCalc tried to access. See [Run a snap under strace](/t/debug-snaps/18420) for more information.
The easiest way to include a library in a snap is to add it to the `stage-packages` of the main build part. Snapcraft will then fetch that library, as well as any other libraries or data that it depends on.

However, in this particular case, Snapcraft's helpfulness leads to duplication, as the dependencies of *libcanberra-pulse* are already provided by the `kde-neon` extension. To avoid this, and keep the size of our snap as small as possible, we can use `override-pull` and `override-build` to make Snapcraft download and install just the library we need.

We're going to do this by adding a new part to *snapcraft.yaml* named `sound-support` as follows:

```yaml
  sound-support:
    plugin: nil
    override-pull: |
      if [ ! -e libcanberra-pulse_*.deb ]; then
        apt-get download libcanberra-pulse:$CRAFT_ARCH_BUILD_FOR
      fi
    override-build: |
      if [ ! -e $CRAFT_PART_INSTALL/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-*/libcanberra-pulse.so ]; then
        dpkg -x $CRAFT_PART_SRC/libcanberra-pulse_*.deb $CRAFT_PART_INSTALL
      fi
```

This part does two things:
- During the *pull* phase, it downloads a package containing the correct version of the *libcanberra-pulse* library for the architecture we're building for using *apt-get*.
- During the *build* phase, it extracts that package to the part's installation folder using *dpkg*, causing the library to be bundled in our snap.

KCalc expects to find the library file (*libcanberra-pulse.so*) and the sound effect (*Oxygen-Sys-App-Message.ogg*) in a particular location. We can add a `layout` to our Snapcraft file to create a link between the *expected* location and the *actual* location within our snap as below. See [Snap layouts](/t/snap-layouts/7207) for more information. 

```yaml
layout:
  /usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so:
    symlink: $SNAP/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so
  /usr/share/sounds/Oxygen-Sys-App-Message.ogg:
    symlink: $SNAP/kf5/usr/share/sounds/freedesktop/stereo/dialog-information.oga
```

If you make these changes to *snapcraft.yaml*, and re-build and reinstall the snap, you should now hear a sound effect when an error occurs.

#### Fixing the cursor

```yaml
    override-prime: |
      craftctl default
      FILE=snap/command-chain/desktop-launch
      if [ -e $CRAFT_PRIME/$FILE ]; then rm $CRAFT_PRIME/$FILE; fi
      CMD='busctl call --user org.freedesktop.portal.Desktop /org/freedesktop/portal/desktop '\
      'org.freedesktop.portal.Settings Read ss org.gnome.desktop.interface'
      sed /wait_for_async_execs/i\
      "export XCURSOR_THEME=\$($CMD cursor-theme | cut -d \\\\\" -f2)\n"\
      "export XCURSOR_SIZE=\$($CMD cursor-size | cut -d' ' -f4)\n" $CRAFT_STAGE/$FILE > $CRAFT_PRIME/$FILE
      chmod +x $CRAFT_PRIME/$FILE 
```

> TO DO: This works on a Ubuntu 24.04 host running Gnome - does it work in KDE and XFCE?

### Complete additional fields

There are various other optional [attributes](/t/8334) that should be completed in *snapcraft.yaml* where possible, such as:
1. a reference to the `license` applicable to your snap
1. the URL of your snap's `website`
1. a `contact` URL or email address
1. a website where users can report any `issues` with your snap (for example, a GitHub or GitLab issues page, or a Discourse forum)
1. a public `source-code` repository where your snap's *snapcraft.yaml* can be found

It's okay to repeat the same URL in different fields, for example if you're running your project entirely out of a GitHub or GitLab repository.

```yaml
license: GPL-2.0-or-later
website: https://github.com/snapcraft-docs/kcalc
contact: https://github.com/snapcraft-docs/kcalc
issues: https://github.com/snapcraft-docs/kcalc/issues
source-code: https://github.com/snapcraft-docs/kcalc
```

## Our updated snap

Here's the code to our updated snap.


## Publishing your snap

To share your snaps you need to publish them in the Snap Store. First, create an account on [the dashboard](https://dashboard.snapcraft.io/dev/account/). Here you can customise how your snaps are presented, review your uploads and control publishing.

You’ll need to choose a unique “developer namespace” as part of the account creation process. This name will be visible by users and associated with your published snaps.

Make sure the `snapcraft` command is authenticated using the email address attached to your Snap Store account:

```bash
$ snapcraft login
```

### Reserve a name for your snap

You can publish your own version of a snap, provided you do so under a name you have rights to. In this case, we'll want to avoid a conflict with the official [`kcalc` snap](https://snapcraft.io/kcalc) published by KDE.

You can register a name on [dashboard.snapcraft.io](https://dashboard.snapcraft.io/register-snap/), or by running the following command:

```bash
$ snapcraft register <your snap name>
```

If needed, update *snapcraft.yaml* so that the `name` of your snap - as well as the name of your (main) app - matches the newly registered name, before rebuilding the snap by running `snapcraft`.

### Upload your snap

Use *Snapcraft* to push the snap to the Snap Store. Assuming you're logged in, the following command would upload our `kcalc-example` snap to the *edge* release channel:

```bash
$ snapcraft upload --release=edge kcalc-example_*.snap
```

If you’re happy with the result, you can commit your *snapcraft.yaml* to a dedicated GitHub repository and [turn on automatic builds](https://build.snapcraft.io) so any further commits automatically are released to the *edge* release channel, without requiring you to manually re-build the snap locally. Once you're happy with the stability of your the snap, you may want to consider promoting it to a *beta*, *candidate* or *stable* release.

Congratulations! You've just built and published your first Qt 5 snap. For a more in-depth overview of the snap building process, see [Creating a snap](/t/6799).
