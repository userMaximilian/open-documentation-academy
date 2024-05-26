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

Typically this guide will take around 20 minutes to follow, and will result in a working Qt 5 and KDE Frameworks application in a snap. Once complete, you'll understand how to package Qt 5 applications as snaps and deliver them to millions of Linux users. After making the snap available in the store, you'll get access to installation metrics and tools to directly manage the delivery of updates to Linux users.

> ⓘ For a brief overview of the snap creation process, including how to install *snapcraft* and how it's used, see [Snapcraft overview](/t/8940). For a more comprehensive breakdown of the steps involved, take a look at [Creating a snap](/t/6799).

## Getting started

For this how-to, we are going to create a snap of KDE's calculator application [KCalc](https://apps.kde.org/kcalc/). We're focusing on version 23.08.5, released in February 2024, as this was the final release to support Qt 5.

Snaps are defined in a single YAML file named *snapcraft.yaml*. Our version of *snapcraft.yaml* is below. We'll break this file down.

[details=Our snapcraft.yaml for KCalc]
```yaml
name: kcalc
adopt-info: kcalc
grade: stable
license: GPL-2.0-or-later
base: core22
confinement: strict

apps:
  kcalc:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions: [kde-neon]
    plugs:
      - audio-playback

slots:
  session-dbus-interface:
    interface: dbus
    name: org.kde.kcalc.desktop
    bus: session

layout:
  /usr/share/applications/org.kde.kcalc.desktop:
    symlink: $SNAP/usr/share/applications/org.kde.kcalc.desktop
  /usr/share/sounds/Oxygen-Sys-App-Message.ogg:
    symlink: $SNAP/kf5/usr/share/sounds/freedesktop/stereo/dialog-information.oga
  /usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so:
    symlink: $SNAP/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-0.30/libcanberra-pulse.so

parts:
  kcalc:
    after: [kde-neon/sdk]
    parse-info: [usr/share/metainfo/org.kde.kcalc.appdata.xml]
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

  sound-support:
    plugin: nil
    override-pull: |
      if [ ! -e libcanberra-pulse_*.deb ]; then apt-get download libcanberra-pulse:$CRAFT_ARCH_BUILD_FOR; fi
    override-build: |
      if [ ! -e $CRAFT_PART_INSTALL/usr/lib/$CRAFT_ARCH_TRIPLET_BUILD_FOR/libcanberra-*/libcanberra-pulse.so ]; then
        dpkg -x $CRAFT_PART_SRC/libcanberra-pulse_*.deb $CRAFT_PART_INSTALL
      fi

```
[/details]

#### Metadata

The `snapcraft.yaml` file starts with a small amount of human-readable metadata. This data is used in the presentation of your app in the Snap Store.

```yaml
name: kcalc
adopt-info: kcalc
grade: stable
license: GPL-2.0-or-later
```

The `name` must be unique in the Snap Store. Valid snap names consist of lower-case alphanumeric characters and hyphens. Names must contain at least one letter and they also cannot start or end with a hyphen. They also cannot be more than 40 characters long.

The `adopt-info` attribute tells Snapcraft that we will make the `title`, `summary`, `description`, `icon` and `version` information available during the `kcalc` build step. (We'll revisit this later on, but for now it's worth bearing in mind that the source code of KCalc includes an [AppStream](https://www.freedesktop.org/wiki/Distributions/AppStream/) metadata file, which we can re-use to avoid duplication in our *snapcraft.yaml*.)

The `grade` is optional, and defines the quality of the snap. We have set the `grade` to `stable` to tell Snapcraft that this snap is suitable for general release, and that it may be published to any release channel. 

The `license` is also optional. As KCalc is [licensed](https://invent.kde.org/utilities/kcalc/-/blob/v23.08.5/README) under the [GNU General Public License 2.0 (or later)](https://invent.kde.org/utilities/kcalc/-/blob/v23.08.5/LICENSES/GPL-2.0-or-later.txt) we have set the `license` to the [SPDX License Expression](https://spdx.org/licenses) `GPL-2.0-or-later`.

#### Base

The `base` keyword defines a special kind of snap that provides a run-time environment with a minimal set of libraries that are common to most applications. Base snaps are transparent to users, but they need to be considered, and specified, when building a snap.

```yaml
base: core22
```

We specified the base as [`core22`](https://snapcraft.io/core22), as it is the only base snap currently supported by the `kde-neon` extension that we use later on.

#### Security model

The `confinement` level defines [how your snap is isolated](/t/6233) from the user's system. 

```yaml
confinement: strict
```
It's worth setting the `confinement` to `devmode` when you start building a new snap, as this mode will help you to work out which [interfaces](t/35928) your snap needs. Once everything is working, and you have defined all the interfaces that your snap needs, you should then change the `confinement` to `strict` mode and check that the snap still works. Snaps with a `confinement` level of `devmode` can't be released in the general release channels of the Snap Store - but those with a confinemnt of `strict` can.

#### Apps

Apps are the commands and services exposed to end users. We define just one app in our snap - with the name `kcalc` - but it's worth noting that snaps can contain multiple apps.

As `kcalc` is the name of both our app and our snap, users will be able run the app directly using its name, i.e. by entering the command `kcalc` in the terminal. 

However, if the names differed, then the command to run our app would be prefixed by the name of the snap. (So, if we had called our snap `kcalc`, but set the app name to `calculator`, then a user would need to run the command `kcalc.calculator` to launch the app from their terminal.) Using the snap name as a prefix ensures that we avoid conflicts with identically named apps defined by other installed snaps, but it isn't always a perfect solution. If you don’t want your app's command to be prefixed, you can request an alias for it on the [Snapcraft forum](https://forum.snapcraft.io/t/455). Any aliases granted by the Snapcraft team will be set up automatically when your snap is installed from the Snap Store.

```yaml
apps:
  kcalc:
    common-id: org.kde.kcalc.desktop
    command: usr/bin/kcalc
    extensions: [kde-neon]
    plugs:
      - audio-playback
```

If your project includes an AppStream metadata file (like KCalc does) then the `common-id` field should be set to the *component identifier* of the relevant component listed in that file. By doing so, we don't need to [manually specify the `.desktop` entry file](/t/13115) using a `desktop` attribute as those details are already defined the in AppStream file. See [Using AppStream metadata](/t/4642#heading--appstream) for more information.

The `command` is the path to the app's executable file, as generated by the relevant build system.

The `kde-neon` extension is essential. It significantly simplifies the building of Qt 5 and KDE Frameworks applications significantly. Among other things:
- it ensures that the relevant Qt/KDE library snaps are available at build time and run time
- it introduces the environment variable `$SNAP_DESKTOP_RUNTIME` to be the location of the Qt/KDE library sna 
- it sets up `plugs` to the `desktop`, `desktop-legacy`, `opengl`, `wayland` and `x11` interfaces
- it also configures the run time environment of the app, so that all desktop functionality is correctly initialised. For further information, see  [The `kde-neon` extension](/t/13752). You can also see how the extension modifies *snapcraft.yaml* by running the command `snapcraft expand-extensions` in your terminal.

Whilst the `kde-neon` extension serves as a great starting point, it often doesn't set up everything that a snap needs. In our case, we have added in a plug to the `audio-playback` interface, so that KCalc is able to play a sound when an error occurs.

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

#### Parts

Parts define how to build your app. We have defined two in our example: `kcalc` and `sound-support`.

```yaml
parts:
  kcalc:
    ...
  sound-support:
    ...
```

The `kcalc` part does the majority of the work: it downloads the KCalc source code together with the required supporting libraries, and then builds the application using [CMake](/t/8621). The `sound-support` part downloads an additional library to make sound effects work. We'll discuss the reasons for splitting the build into two parts below.


##### The *kcalc* part

[quote]
:construction: TO DO
[/quote]

```yaml

parts:
  kcalc:
    after: [kde-neon/sdk]
    parse-info: [usr/share/metainfo/org.kde.kcalc.appdata.xml]
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
[quote]
:construction: TODO
- `after`
- `parse-info` points to the location of the AppStream metadata file as 'installed' in our snap. As we used `adopt-info: kcalc` in the initial metadata section of our snap, the relevant fields from the AppStream file will be used to fill in the `title`, `summary`, `description`, `icon` and `version` number of this snap.
- [The CMake plugin](/t/the-cmake-plugin/8621) then uses `cmake` to build the part. 
- `build-packages`. We don't specify and `build-snaps` ourselves, as we rely on the `kde-neon` extension to ensure that the relevant Qt/KDE development libraries are available during the build.
- `stage-packages` are the packages required by KCalc to run, and mirror the same packages required by the binary on a standard distribution installation.
- `source`
- `cmake-params`
- `override-build`
- `override-prime`
[/quote]

##### The *sound-support* part

[quote]
:construction: TODO
- plugin
- override-pull
- override-build
[/quote]

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

## Publishing your snap

To share your snaps you need to publish them in the Snap Store. First, create an account on [the dashboard](https://dashboard.snapcraft.io/dev/account/). Here you can customise how your snaps are presented, review your uploads and control publishing.

You’ll need to choose a unique “developer namespace” as part of the account creation process. This name will be visible by users and associated with your published snaps.

Make sure the `snapcraft` command is authenticated using the email address attached to your Snap Store account:

```bash
$ snapcraft login
```

### Reserve a name for your snap

You can publish your own version of a snap, provided you do so under a name you have rights to. In this case, we would need to rename our snap from *kcalc* to something that doesn't conflict with the existing KDE snap (e.g. `kcalc-unofficial`)

You can register a name on [dashboard.snapcraft.io](https://dashboard.snapcraft.io/register-snap/), or by running the following command:

```bash
$ snapcraft register mysnap
```

Be sure to update the `name:` in your *snapcraft.yaml* to match this registered name - and the name of the app too, if needed - then run `snapcraft` again.

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

### Upload your snap

Use *Snapcraft* to push the snap to the Snap Store. Assuming you're logged in, the following command would upload a snap to the *edge* release channel:

```bash
$ snapcraft upload --release=edge <your snap file>.snap
```

If you’re happy with the result, you can commit the *snapcraft.yaml* to a dedicated GitHub repository and [turn on automatic builds](https://build.snapcraft.io) so any further commits automatically are released to the *edge* release channel, without requiring you to manually re-build the snap locally. Once you're happy with the stability of your the snap, you may want to consider promoting it to a *beta*, *candidate* or *stable* release.

Congratulations! You've just built and published your first Qt 5 snap. For a more in-depth overview of the snap building process, see [Creating a snap](/t/6799).
