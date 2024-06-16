## Why are snaps good for Qt 5 and KDE Frameworks applications?

Here are just some of the many reasons why you should consider packaging and distributing your Qt 5 and KDE Frameworks applications as snaps:

* **Snaps are easy to create**\
  The process of creating a snap for a Qt 5 and/or KDE Frameworks application builds on standard tools like `cmake` and `make`, enabling you to re-use the application's existing build system when creating your snap.
* **Snaps are easy to discover and install**\
  Millions of users can browse and install snaps in the online [Snap Store](https://snapcraft.io/store), in the [official application](https://snapcraft.io/snap-store) and other graphical store frontends such as GNOME Software or KDE Discover, or from the command line.
* **Snaps install and run the same across Linux**\
  You can rely on carefully curated extensions to provide the main libraries that your applications depend on, and then bundle the exact versions of any additional dependencies within the snap.
* **You control the release schedule**\
  You decide when a new version of your application is released without having to wait for distributions to catch up.
* **Snaps automatically update to the latest version**\
  Users' systems check for new versions and upgrade in the background four times a day.
* **Upgrades are safe**\
  If your snap fails to upgrade, users automatically roll back to the previous revision.
* **Snaps run in a sandbox by default**\
  Most snaps run in an isolated environment and only have access to resources like user files and networking via defined interfaces, enhancing security, system stability and user trust.

### Build a snap in 20-30 minutes

This guide should take around 20-30 minutes to follow. Once complete, you'll understand how to package Qt 5 and KDE Frameworks applications as snaps and deliver them to millions of Linux users. After making your snap available in the Snap Store, you'll get access to installation metrics and tools to directly manage the delivery of updates to Linux users.

> ⓘ For a brief overview of the snap creation process, including how to install Snapcraft and how it's used, see [Snapcraft overview](/t/snapcraft-overview/8940). For a more comprehensive breakdown of the steps involved, take a look at [Creating a snap](/t/creating-a-snap/6799).

## Getting started

Snaps are defined in a single YAML file named *snapcraft.yaml* located in the root folder of your project.

We will start by creating a short *snapcraft.yaml* file that builds a snap of KDE's calculator application [KCalc](https://apps.kde.org/kcalc/). We will then improve our *snapcraft.yaml*, before exploring how you can submit your own snaps to the Snap Store.

The initial version of our *snapcraft.yaml* file can be found below. We'll explain the contents of this file in the following sections of this guide.

[details=Click here to expand/collapse our initial *snapcraft.yaml*]
```yaml
name: kcalc-example
adopt-info: kcalc
grade: devel

base: core22
confinement: devmode

apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions:
      - kde-neon

parts:
  kcalc:
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    parse-info:
      - usr/share/metainfo/org.kde.kcalc.appdata.xml
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    plugin: cmake
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"
```
[/details]

#### Metadata

The *snapcraft.yaml* file starts with a small amount of human-readable metadata. This data determines how the snap is presented in the Snap Store, so you should take care to ensure that it is complete and correct prior to submission. See [Snapcraft top-level metadata](/t/snapcraft-top-level-metadata/8334) for more information.

```yaml
name: kcalc-example
adopt-info: kcalc
grade: devel
```

Every snap needs a `name`. Valid names consist of up to 40 lower-case ASCII letters, numbers and hyphens. Names must contain at least one letter, and they cannot start or end with a hyphen. If you want to publish your snap, you'll need to specify a `name` that hasn't already been taken in the Snap Store.

You also need to:
1. include a short `summary` of what your snap does, a longer `description` and the `version` number of your snap; or
2. use the `adopt-info` attribute to tell Snapcraft that this information will be defined during the build (e.g. by a script, or in an [AppStream metadata file](/t/using-external-metadata/4642)).
We're opting to use `adopt-info` for our example snap, as the KCalc source code includes a suitable AppStream file.

The `grade` is optional. It defines the quality of the snap. We're setting the `grade` as `devel` to tell Snapcraft that our snap is in development, and that it isn't yet suitable to be published to the `stable` or `candidate` release [channels](/t/channels/551) of the Snap Store.

#### Base

The `base` keyword defines a special kind of snap that provides a run-time environment with a minimal set of libraries that are common to most applications. Bases are transparent to users, but they need to be considered, and specified, when building a snap. See [Base snaps](/t/base-snaps/11198) for more information.

```yaml
base: core22
```

[`core22`](https://snapcraft.io/core22) is a standard base for snap building and is based on [Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/).

#### Security model

The `confinement` defines how [isolated](/t/snap-confinement/6233) your snap is from the user's system.

```yaml
confinement: devmode
```

It's worth setting the `confinement` to `devmode` when you start working on a new snap. `devmode` runs your application without the security confinement imposed by the [snap daemon](/t/installing-snapd/6735). It also helps you to identify the [interfaces](/t/supported-interfaces/7744) that your snap needs, helping to speed up the snap building process. See [Debug snaps](/t/debugging-snaps/18420) for more information.

Once your snap is ready, you'll need to change the `confimenent` to `strict` and test that the snap still works, before you make it generally available to users of the Snap Store. (`devmode` snaps may only be released to the hidden "edge" channel of the Snap Store where you and other developers can install them.)

#### Apps

*Apps* are the commands and services that you want your users to run or use.

For the purposes of KCalc, we only need to define one app, which we're naming `kcalc-example`. However, it's worth noting that a single snap can contain multiple apps.

As `kcalc-example` is both the name of our *app* and the name of our *snap*, users can launch the app by simply running the command `kcalc-example`.

If we used different names for the *app* and the *snap*, then the command would instead take the form `<app name>.<snap name>`. (For example, if we kept our snap's name as `kcalc-example`, but changed the app name to `calculator`, then the command to run our app would be `kcalc-example.calculator`.) The use of a prefix helps us to avoid conflicts with apps defined by other installed snaps. If you don't want your users to have to specify a prefix, you can apply for a command alias on the [Snapcraft forum](https://forum.snapcraft.io/t/455). Any aliases approved by the Snap Store's review team will be set up automatically when a user installs your snap from the Snap Store.

```yaml
apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions: 
      - kde-neon
```

The `common-id` field tells Snapcraft where to find information about our app (such as the location of a [desktop entry file](/t/desktop-menu-support/13115) which would normally be specified with a `desktop` entry) within an AppStream metadata file. See [Using AppStream metadata](/t/4642#heading--appstream) for more information.

The `command` is the path to the app's main executable file within the snap. 

The [`kde-neon` extension](/t/the-kde-neon-extension/13752) ensures that the relevant Qt/KDE platform snaps are available during the build process and at run time. The extension also:
- adds `plugs` to the `desktop`, `desktop-legacy`, `opengl`, `wayland` and `x11` interfaces; and
- configures the run-time environment of the app, ensuring that all relevant desktop functionality is correctly initialised.

#### Parts

Parts define how to build your app. We're defining just one part for now, which we name `kcalc`.

```yaml
parts:
  kcalc:
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    parse-info:
      - usr/share/metainfo/org.kde.kcalc.appdata.xml
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    plugin: cmake
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"
```

`source` tells Snapcraft where to find the source code for KCalc. Whilst we're using the URL of a tarball file hosted on a website, Snapcraft can also use source code from a version control system or a local directory.

> ⓘ We're using version 23.08.5 of KCalc released in February 2024 for this how-to. At the time of writing (June 2024) this is the latest - and possibly final - version to support Qt 5.

We set `parse-info` to the location of the AppStream metadata file as it will be installed in our snap. This file provides the information needed by `adopt-info` and `common-id` that we added earlier on.

In addition to Qt 5 and the KDE Frameworks, KCalc depends on two libraries: MPFR and GMP. We use:
- `build-packages` to ensure that the *development* versions of these libraries are during the build process, and
- `stage-packages` to bundle the *run-time* versions within the snap.
The packages will be obtained from the Ubuntu [package archive](https://packages.ubuntu.com) applicable to the chosen `base`. (In our case, as we're using `core22`, the packages need to be available in *Ubuntu 22.04 LTS*.) We don't need to list any of the Qt or KDE libraries, as the `kde-neon` extension makes them available for us.

[The `cmake` `plugin`](/t/the-cmake-plugin/8621) tells Snapcraft to build the KCalc source code using the *CMake build system*.

We pass various build-modifying parameters to CMake using `cmake-parameters`. For example:
- `"-DINSTALL_ICONS=ON"` embeds KCalc's icons within the snap, so that they correctly appear in the user's application launcher. If this parameter is omitted, then users are likely to see a default icon instead.
- `"-DKF5DocTools_FOUND=OFF"` disables the generation of KCalc's documentation files. This helps to speed up the build process and reduce the size of our snap. Users can still access the documentation online via KCalc's *Help* menu.

### Building the snap

You can download the repository for our example KCalc snap by running the following command:

```bash
git clone https://github.com/snapcraft-docs/kcalc-example
```

The *snapcraft.yaml* file that we've discussed so far can be found in the *initial* directory.

You can build the snap by simply executing `snapcraft` in that directory. The build should take about 5 to 10 minutes to complete. Snapcraft will display various status messages in your terminal whilst the build is ongoing. If all goes well, you should see something like the following once the build is complete:

```bash
$ snapcraft
Generated snap metadata
Created snap package kcalc-example_23.08.5_amd64.snap
```

> ⓘ Although we have only tested this example on an *amd64* system, the `kde-neon` extension is also available for *arm64*.

The resulting snap can be installed locally. You need to specify the `--devmode` option as to install an unconfined snap, like our KCalc example:

```bash
$ sudo snap install kcalc-example_*.snap --devmode
```

You can then try it out by running one of the following commands:

```bash
$ snap run kcalc-example
```
```bash
$ kcalc-example
```

Removing the snap is simple too:

```bash
$ sudo snap remove kcalc-example
```

You can clean up the build environment with the following command:

```bash
$ snapcraft clean
```

By default, when you make a change to *snapcraft.yaml*, Snapcraft only builds the parts that have changed. Cleaning a build, however, forces your snap to be rebuilt in a clean environment and will take longer.

### Improving the snap

You might notice a few issues when testing our snap, for example:
- error sounds might not play
- the application might use different cursors to the rest of your applications

Let's try to improve the user experience by addressing them:

[details=Error sounds]
KCalc attempts to play a sound when an error occurs, for example if a user tries to add multiple decimal places to a number. We'll need to grant KCalc access to the correct audio interface, and make sure that it has access to the correct libraries and assets, before it can play these sounds successfully.

To start with, we need to update *snapcraft.yaml* to grant our snap access to the `audio-playback` interface, as this isn't one of the interfaces that the `kde-neon` extension adds for us. We can do this by adding a `plugs` entry with the value `audio-playback` to the `kcalc-example` app definition. The resulting `apps` section looks like this:

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

KCalc plays error sounds using KDE's *KNotification* Framework, which in turn relies upon the *libcanberra* audio system. Whilst our application has access to the main *libcanberra* library, it doesn't have access to the *libcanberra-pulse* plugin, so we need to bundle it with our snap.

The easiest way to bundle a library with a snap is to add it to the `stage-packages` section of the main build part. Snapcraft will then fetch that library, as well as any other libraries or data that it depends on, at the start of the build.

This behaviour is usually very helpful, but in our case it leads to unwanted duplication, as the dependencies of *libcanberra-pulse* are already bundled in the platform snap provided by the `kde-neon` extension. To avoid this, and keep the size of our snap as small as possible, we can use `override-pull` and `override-build` to make Snapcraft download and install just the library we need.

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

This part performs two actions:
- During the *pull* phase, it downloads a package containing the correct version of the *libcanberra-pulse* library for the architecture we're building for using *apt-get*.
- During the *build* phase, it extracts that package to the part's installation folder using *dpkg*, causing the library to be bundled in our snap.

As we specified the `plugin` as `nil`, Snapcraft will only perform the actions that we specify in `override-pull` and `override-build`.

Finally, KCalc expects to find the library file (*libcanberra-pulse.so*) and the sound effect (*Oxygen-Sys-App-Message.ogg*) in particular locations, which differ from where the files are actually stored in our snap. We can add a `layout` section to our Snapcraft file to link between the *expected* and *actual* locations as below. See [Snap layouts](/t/snap-layouts/7207) for more information. 

```yaml
layout:
  /usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so:
    symlink: $SNAP/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so
  /usr/share/sounds/Oxygen-Sys-App-Message.ogg:
    symlink: $SNAP/kf5/usr/share/sounds/freedesktop/stereo/dialog-information.oga
```

If you make these changes to *snapcraft.yaml*, and then re-build and reinstall the snap, you should now hear a sound when an error occurs.
[/details]

[details=Cursor themes]
If you run our `kcalc-example` snap under [*Wayland*](https://wayland.freedesktop.org), you might experience KCalc using a different cursor theme to the rest of your applications.

This is because Qt 5 applications on Wayland expect to learn about the cursor theme and size from two environment variables: `XCURSOR_THEME` and `XCURSOR_SIZE`. However, not all desktop environments set these values, meaning that applications like KCalc can't determine which cursor theme or size to use, so they fall back to default settings instead.

We can try to resolve this issue for users of the GNOME desktop environment on Wayland by inserting a few lines into the `kde-neon` *launcher script* that sets up the run-time environment when our snap is launched. We'll do this with a new part named `set-cursor-variables`:

```yaml
  set-cursor-variables:
    after:
      - kde-neon/sdk
    source: https://github.com/snapcraft-docs/kcalc-example.git
    source-type: git
    source-subdir: patches
    plugin: dump
    override-prime: |
      FILE=snap/command-chain/desktop-launch
      if [ -e "$CRAFT_PRIME/$FILE" ]; then rm "$CRAFT_PRIME/$FILE"; fi
      cp "$CRAFT_STAGE/$FILE" "$CRAFT_PRIME/$FILE"
      patch -p0 < "$CRAFT_STAGE"/set-cursor-variables.patch
```

We use the `after` attribute to ensure that our modifications are only applied after the *launcher script* is generated by the (hidden) `kde-neon/sdk` build step. The various `source` lines tell Snapcraft how to fetch a community-contributed patch file named [`set-cursor-variables.patch`](https://github.com/snapcraft-docs/kcalc-example/blob/patch/set-cursor-variables.patch) from our GitHub project repository. The `override-prime` step then updates the `desktop-launch` script using the amendments listed in our patch file.

[note]

The patch only makes minimal changes to our snap. It isn't intended to be a comprehensive solution that works with every desktop environment or with every icon theme. As our snap cannot access the host system's cursor store, it relies on the user's chosen theme being contained in the `gtk-common-themes` content snap or in the Qt/KDE Frameworks platform snap. If the user uses a different theme, then the snap will fall back to a default cursor.

If you have any comments or suggestions, please let us know in the [Snapcraft forum post](https://forum.snapcraft.com/t/qt5-and-kde-frameworks-applications/13753) for this document.
[/note]

[/details]

There are a few other changes that we can make before we publish our snap:

[details=Complete additional fields]
There are various optional [attributes](/t/snapcraft-top-level-metadata/8334) that should be added to *snapcraft.yaml* where relevant such as:
1. the name of the `license` applicable to your snap
1. the URL of your snap's `website`
1. a `contact` URL or email address
1. a website where users can report any `issues` with your snap (for example, a GitHub or GitLab issues page, or a Discourse forum)
1. a public `source-code` repository where your snap's *snapcraft.yaml* can be found
1. a `title` for use by store frontends like Ubuntu's App Centre
1. the `icon` that we want users to see in the Snap Store or in store frontends

It's okay to repeat the same URL in different fields, for example if you're running your project entirely out of a GitHub or GitLab repository. We're doing so for `kcalc-example`, as follows:

```yaml
license: GPL-2.0-or-later
website: https://github.com/snapcraft-docs/kcalc
contact: https://github.com/snapcraft-docs/kcalc
issues: https://github.com/snapcraft-docs/kcalc/issues
source-code: https://github.com/snapcraft-docs/kcalc
```
[/details]

[details=Update the confinement to `strict`]
As we're happy that our *snapcraft.yaml* now refers to all the instances it needs to function properly, now is a good time to update the `confinement` from `devmode` to `strict`, before carrying out further testing.
[/details]

### Our updated snap

Here's the updated *snapcraft.yaml* for our snap. If you cloned the *kcalc-example* repository from GitHub earlier on, you should find a copy in the *revised* folder of that repository.

[details=Click here to expand our updated *snapcraft.yaml*]
```
name: kcalc-example
adopt-info: kcalc
grade: devel

license: GPL-2.0-or-later
website: https://github.com/snapcraft-docs/kcalc-example/
contact: https://github.com/snapcraft-docs/kcalc-example/
issues: https://github.com/snapcraft-docs/kcalc-example/issues/
source-code: https://github.com/snapcraft-docs/kcalc-example/

base: core22
confinement: strict

apps:
  kcalc-example:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions:
      - kde-neon
    plugs:
      - audio-playback

parts:
  kcalc:
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    parse-info:
      - usr/share/metainfo/org.kde.kcalc.appdata.xml
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    plugin: cmake
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"

  sound-support:
    plugin: nil
    override-pull: |
      if [ ! -e libcanberra-pulse_*.deb ]; then
        apt-get download libcanberra-pulse:$CRAFT_ARCH_BUILD_FOR;
      fi
    override-build: |
      if [ ! -e $CRAFT_PART_INSTALL/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-*/libcanberra-pulse.so ]; then
        dpkg -x $CRAFT_PART_SRC/libcanberra-pulse_*.deb $CRAFT_PART_INSTALL
      fi

  set-cursor-variables:
    after:
      - kde-neon/sdk
    source: https://github.com/snapcraft-docs/kcalc-example.git
    source-type: git
    source-depth: 1
    source-subdir: patches
    plugin: dump
    override-prime: |
      FILE=snap/command-chain/desktop-launch
      if [ -e "$CRAFT_PRIME/$FILE" ]; then rm "$CRAFT_PRIME/$FILE"; fi
      cp "$CRAFT_STAGE/$FILE" "$CRAFT_PRIME/$FILE"
      patch -p0 < "$CRAFT_STAGE"/set-cursor-variables.patch

layout:
  /usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so:
    symlink: $SNAP/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so
  /usr/share/sounds/Oxygen-Sys-App-Message.ogg:
    symlink: $SNAP/kf5/usr/share/sounds/freedesktop/stereo/dialog-information.oga
```
[/details]

## Publishing your snap

To share your snaps you need to publish them in the Snap Store. First, create an account on [the dashboard](https://dashboard.snapcraft.io/dev/account/). Here you can customise how your snaps are presented, review your uploads and control publishing.

You’ll need to choose a unique "developer namespace" as part of the account creation process. This name will be visible by users and associated with your published snaps.

Make sure the `snapcraft` command is authenticated using the email address attached to your Snap Store account:

```bash
$ snapcraft login
```

### Reserve a name for your snap

You can publish your own version of a snap, provided you do so under a name that's appropriate. For example, if we wanted to publish our KCalc snap, we would need to pick a name that doesn't conflict or cause any confusion with the official [`kcalc` snap published by KDE](https://snapcraft.io/kcalc).

You can register a name on [dashboard.snapcraft.io](https://dashboard.snapcraft.io/register-snap/) or by running the following command:

```bash
$ snapcraft register <snap name>
```

Be sure to update the `name` of your snap - as well as the name of your (main) app - matches the newly registered name, before rebuilding the snap by running `snapcraft`.

### Upload your snap

Use `snapcraft` to push the snap to the Snap Store. The following command would upload our `kcalc-example` snap to the *edge* release channel:

```bash
$ snapcraft upload --release=edge kcalc-example_*.snap
```

If you’re happy with the result, you should consider committing the *snapcraft.yaml* to a GitHub repository and [turn on automatic builds](https://build.snapcraft.io) so any further commits are built and released to the *edge* release channel automatically. Once you're happy with the quality of your snap, you may want to consider promoting it to a *beta*, *candidate* or *stable* release.

Congratulations! You've just built and published your first Qt 5 snap. For a more in-depth overview of the snap building process, see [Creating a snap](/t/creating-a-snap/6799).
