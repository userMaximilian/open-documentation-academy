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

> ⓘ For a brief overview of the snap creation process, including how to install `snapcraft` and the role of *snapcraft.yaml* in defining a snap, see [Snapcraft overview](/t/snapcraft-overview/8940). For a more comprehensive breakdown of the steps involved, take a look at [Creating a snap](/t/creating-a-snap/6799).

We'll start by preparing a *snapcraft.yaml* file that builds a *mostly* working development version of KCalc. We'll then look at a few ways in which we can improve our *snapcraft.yaml* and submit our snap to the Snap Store.

Typically this guide will take around 20 minutes to follow. Once complete, you'll understand how to package Qt 5 applications as snaps and deliver them to millions of Linux users. After making the snap available in the store, you'll get access to installation metrics and tools to directly manage the delivery of updates to Linux users.

## Getting started

The full version of our *snapcraft.yaml* file is below. We'll break this file down.

[details=Our initial snapcraft.yaml for KCalc]
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
    parse-info: [ usr/share/metainfo/org.kde.kcalc.appdata.xml ]
    plugin: cmake
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
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

[note]If you want to snap an application that doesn't include an external metadata file, then you should delete the `adopt-info` line and add separate entries for the `summary`, `description`, `version`, `title` and `icon` attributes instead.[/note]

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
    parse-info: [ usr/share/metainfo/org.kde.kcalc.appdata.xml ]
    plugin: cmake
    build-packages:
      - libmpfr-dev
      - libgmp-dev
    stage-packages:
      - libmpfr6
      - libgmp10
    source: https://invent.kde.org/utilities/kcalc/-/archive/v23.08.5/kcalc-v23.08.5.tar.gz
    cmake-parameters:
      - "-DCMAKE_BUILD_TYPE=Release"
      - "-DCMAKE_INSTALL_PREFIX=/usr"
      - "-DKDE_INSTALL_USE_QT_SYS_PATHS=ON"
      - "-DKDE_SKIP_TEST_SETTINGS=ON"
      - "-DINSTALL_ICONS=ON"
      - "-DKF5DocTools_FOUND=OFF"
```

We set `parse-info` to the location of the AppStream metadata file in our snap. This file will be used by the `adopt-info` and `common-id` attributes discussed above.

[The `cmake` `plugin`](/t/the-cmake-plugin/8621) tells Snapcraft to use the *CMake build system* to build the part. 
In addition to Qt 5 and the KDE Frameworks, KCalc depends on two libraries: libmpfr and libgmp. We instruct Snapcraft to obtain the *development* versions of these libraries under `build-packages`, and to install the run time versions within the snap itself under `stage-packages`.

We don't need to specify the Qt or KDE libraries here, as the `kde-neon` extension handles this for us.

`source` tells Snapcraft where the source code for the part is located. Whilst we're linking to a compressed tarball hosted on a public website, Snapcraft can also retrieve source code from a version control system or a local directory. 

[Note] We're specifically using version 23.08.5 released in February 2024 for this how-to. This is the latest version that supports Qt 5 at the time of writing (May 2024). [/note]

We can pass various parameters to CMake using `cmake-parameters`. These are mostly self-explanatory (i.e. building in 'release' mode, defining the locations for the build and skipping build tests) however it's worth briefly discussing the final two:
- `"-DINSTALL_ICONS=ON"` embeds the KCalc icons within the snap, so that it correctly appears in the user's application launcher (and the Snap Store). If this parameter is omitted, then you may see a default icon instead.
- `"-DKF5DocTools_FOUND=OFF"` disables the generation of documentation for KDE applications. This slightly reduces the size of our snap, and users are not significantly disadvantaged as they can still access the online version of the KCalc manual from the application's *Help* menu.


### Building the snap

You can download the example *snapcraft.yaml* file using Git with the following command:

```bash
$ git clone https://github.com/snapcraft-docs/kcalc.git
```

Alternatively, make a new directory and download the file using `wget` or `curl`, e.g.:

```bash
$ wget https://github.com/snapcraft-docs/kcalc/raw/snapcraft.yaml
```

After you've downloaded *snapcraft.yaml*, you can build the snap by simply executing the *snapcraft* command in the project directory. It should take about five to ten minutes to complete the process. You'll see *snapcraft* display various messages regarding the status of the build as it's happening. If all goes well, once the command has finished you should see something like the following:

```bash
$ snapcraft
Generated snap metadata
Created snap package kcalc_23.08.5_amd64.snap
```

> :warning: The extension used in this example has only been tested on an amd64 system. The relevant packages are available for arm64, but we haven't tested this.

The resulting snap can be installed locally. This requires the `--dangerous` flag because the snap is not signed by the Snap Store.

```bash
$ sudo snap install kcalc_23.08.5_amd64.snap --dangerous
```

You can then try it out:

```bash
$ snap run kcalc
```

Or, if you don't have kcalc installed via some other means (e.g. using your system's package manager), simply by running `kcalc`:

```bash
$ kcalc
```

Removing the snap is simple too:

```bash
$  sudo snap remove kcalc
```

You can clean up the build environment with the following command:

```bash
$ snapcraft clean
```

By default, when you make a change to *snapcraft.yaml*, Snapcraft will only build the parts that have changed. Cleaning a build forces your snap to be rebuilt in a clean environment and will take longer.


#### Improving our snap

If you have tested the snap, you might have noticed a few things:
- the cursor might not match your environment
- sounds might not play


Whilst the `kde-neon` extension does the majority of the setup that a Qt 5 application needs ... it often doesn't set up everything needed by a Qt 5 snap. In our case, we have added in a plug to the `audio-playback` interface, so that KCalc is able to play a sound when an error occurs.




In addition, there is 

[quote]
:construction: TODO
- `after`
- `override-build`
- `override-prime`





The `license` is also optional. As KCalc is licensed under the *GNU General Public License 2.0* (or later) we're setting the `license` to the corresponding [SPDX License Expression](https://spdx.org/licenses) `GPL-2.0-or-later`.






#### kcalc part revisited

```yaml
    override-build: |
      if [ ! -e /usr/share/xml ]; then
        ln -s /snap/kf5-*-core22-sdk/current/usr/share/xml/ /usr/share/xml
      fi
      craftctl default
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


#### Slots

```yaml
slots:
  session-dbus-interface:
    interface: dbus
    name: org.kde.kcalc.desktop
    bus: session
```

KDE and other desktop applications often communicate with each other using the D-Bus protocol. The section above sets up our snap to listen on the per-user `session` bus using the name `org.kde.kcalc.desktop`. Other snaps can connect to our snap using the `plugs` and `slots` [interface management](/t/6154) so that they can communicate with each other. However, this connection needs to be performed by the user manually (e.g. by issuing a `snap connect` command in the terminal) or the connecting snap needs to be granted [permission to auto-connect](/t/1822).

For more information about D-Bus, see [The dbus interface](/t/2038).

#### Layout

```yaml
layout:
  /usr/share/applications/org.kde.kcalc.desktop:
    symlink: $SNAP/usr/share/applications/org.kde.kcalc.desktop
  /usr/share/sounds/Oxygen-Sys-App-Message.ogg:
    symlink: $SNAP/kf5/usr/share/sounds/freedesktop/stereo/dialog-information.oga
  /usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so:
    symlink: $SNAP/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so
```

We use a `layout` to create symbolic links to resources that KCalc expects to see in a particular location in /usr. For example, the second entry ensures that KCalc can find a sound effect present in the Qt/KDE run time snap, whilst the third entry enables KCalc to find an audio library that we add to our snap during the build process.

##### The *sound-support* part

[quote]
:construction: TODO
- plugin
- override-pull
- override-build
[/quote]

### Complete additional fields

There are various other [attributes](/t/8334) that, although optional, should be completed in your *snapcraft.yaml* file where possible. For example:
1. a general purpose `contact` URL or email address
1. the URL of your the snap project's `website`
1. the URL of the `source-code` repository where your snap's *snapcraft.yaml* is hosted
1. the URL of a website where `issues` can be reported with your snap (e.g. a GitHub or GitLab issues page, or a Discourse forum)

There's no problem with repeating the same URL where needed. For our KCalc example, if we simply wanted to direct everyone to the GitHub repository, we could simply add the following to the start of *snapcraft.yaml*:

```yaml
contact: https://github.com/snapcraft-docs/kcalc
website: https://github.com/snapcraft-docs/kcalc
source-code: https://github.com/snapcraft-docs/kcalc
issues: https://github.com/snapcraft-docs/kcalc/issues
```











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

If you’re happy with the result, you can commit the *snapcraft.yaml* to a dedicated GitHub repository and [turn on automatic builds](https://build.snapcraft.io) so any further commits automatically are released to the *edge* release channel, without requiring you to manually re-build the snap locally. Once you're happy with the stability of your the snap, you may want to consider promoting it to a *beta*, *candidate* or *stable* release.

Congratulations! You've just built and published your first Qt 5 snap. For a more in-depth overview of the snap building process, see [Creating a snap](/t/6799).
