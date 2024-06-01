The `kde-neon` extension helps with the creation of snaps that use [Qt 5](https://doc.qt.io/qt-5/) and/or [the KDE Frameworks](https://kde.org/products/frameworks/).

[note]

Snapcraft extensions enable snap developers to easily incorporate a set of common requirements into a snap. See [Snapcraft extensions](/t/snapcraft-extensions/13486) for further details.
[/note]

The extension is currently available for `core22`, `core20` and `core18`. New snaps should be built using `core22`, as the `core20` and `core18` extensions are no longer being actively developed (and as `core18` itself [has been deprecated](/t/base-snaps/11198#heading--deprecated)).

The current versions of Qt 5 and the KDE Frameworks for each base are:

| Base | Versions | Platform snap | Build snap | Released
|-|---|---|---|---|
| core22 | Qt 5.15.11 and KF 5.113 | [kf5-5-113-qt-5-15-11-core22](https://snapcraft.io/kf5-5-113-qt-5-15-11-core22) | [kf5-5-113-qt-5-15-11-core22-sdk](https://snapcraft.io/kf5-5-113-qt-5-15-11-core22-sdk) | Snapcraft 8.1.0 |
| core20 | Qt 5.15.7 and KF 5.99 | [kde-frameworks-5-99-qt-5-15-7-core20](https://snapcraft.io/kde-frameworks-5-99-qt-5-15-7-core20) |  [kde-frameworks-5-99-qt-5-15-7-core20-sdk](https://snapcraft.io/kde-frameworks-5-99-qt-5-15-7-core20-sdk) | Snapcraft 7.3 |
| core18 | Qt 5.14.1 and KF 5.67 | [kde-frameworks-5-core18](https://snapcraft.io/kde-frameworks-5-core18) | [kde-frameworks-5-core18-sdk](https://snapcraft.io/kde-frameworks-5-core18-sdk) | 14 February 2023 |

[note type="caution"]

The extension is designed for C++-based applications. It does not provide the bindings needed for *Qt for Python* (PySide2) applications.

In addition, the extension does not cover include every optional Qt library: for example, it does not include Qt3D, QtCharts, QtDataVisualization or QtGamepad.
[/note]

The remainder of this document will focus on the current version of the `core22` extension.

## How to use it

To use the extension, add an `extensions` parameter with the value `kde-neon` to the `apps` definition in your `snapcraft.yaml` file. 

```yaml
apps:
  kcalc-example:
    command: usr/bin/kcalc
    extensions:
      - kde-neon
    ...
```

See [Qt 5 and KDE Frameworks applications](/t/qt5-and-kde-frameworks-applications/13753) for an overview of using this extension to build a Qt 5 application.

## Interface connections

The extension connects your snap to the following content snaps:

- [`kf5-5-113-qt-5-15-11-core22`](https://snapcraft.io/kf5-5-113-qt-5-15-11-core22) for the Qt 5 and KDE Frameworks run-time libraries (Snapcraft 8.1.0 onwards)
- [`gtk-common-themes`](https://snapcraft.io/gtk-common-themes) for common icon, cursor and sound themes.

This is achieved by adding the following plugs to your *snapcraft.yaml*:
```yaml
plugs:
    desktop:
        mount-host-font-cache: false
    icon-themes:
        interface: content
        target: $SNAP/data-dir/icons
        default-provider: gtk-common-themes
    sound-themes:
        interface: content
        target: $SNAP/data-dir/sounds
        default-provider: gtk-common-themes
    kf5-5-113-qt-5-15-11-core22:
        content: kf5-5-113-qt-5-15-11-core22-all
        interface: content
        default-provider: kf5-5-113-qt-5-15-11-core22
        target: $SNAP/kf5
```

In addition, the extension adds the following plugs to your application:

```yaml
apps:
  <each application>:
    plugs:
      - desktop
      - desktop-legacy
      - opengl
      - wayland
      - x11
```

See [Adding interfaces](/t/adding-interfaces/13123) for more details.

## Included packages

The `kde-neon` extension is derived from two separate snaps: a build snap and a platform/run-time snap.

The _build snap_ ensures that the relevant Qt 5 and KDE Frameworks development libraries (and supporting files) are available to Snapcraft during the build process. These libraries are sourced from the Ubuntu-based *KDE neon* Linux distribution, which provides more much recent versions of Qt 5 and the KDE Frameworks than are available in the Ubuntu 22.04 / `core22` software repositories.

The _platform snap_ makes the corresponding run-time libraries available to your snap when it is launched by your users.

## Environment variables

In addition to using the *build* and *platform* snaps, the `kde-neon` extension also sets a collection of environment variables, links, default plugs for the app to use, and a default `build-environment` for each part in your snap to use.

### Build variables

The following `build-environment` section is made available to each part built in your snap. If you define other `build-environment` variables, then those will get added to these and the combined set will be used. If you define another value for one of these variables, then the value you've defined will be used instead of the value defined by the extension.

```yaml
build-environment:
-   PATH: /snap/kf5-5-113-qt-5-15-11-core22-sdk/current/usr/bin${PATH:+:$PATH}
-   XDG_DATA_DIRS: $CRAFT_STAGE/usr/share:/snap/kf5-5-113-qt-5-15-11-core22-sdk/current/usr/share:/usr/share${XDG_DATA_DIRS:+:$XDG_DATA_DIRS}
-   SNAPCRAFT_CMAKE_ARGS: -DCMAKE_FIND_ROOT_PATH=/snap/kf5-5-113-qt-5-15-11-core22-sdk/current${SNAPCRAFT_CMAKE_ARGS:+:$SNAPCRAFT_CMAKE_ARGS}
```

### Run time variables

The following environment is set when your application is run:
```yaml
environment:
  SNAP_DESKTOP_RUNTIME: $SNAP/kf5
```

In addition, the extension adds a build part named `kde-neon/sdk` that assembles the following shell scripts into a single script named `desktop-launch`:
- [desktop-exports](https://github.com/canonical/snapcraft/blob/main/extensions/desktop/common/desktop-exports)
- [launcher-specific](https://github.com/canonical/snapcraft/blob/main/extensions/desktop/kde-neon/launcher-exec)
- [mark-and-exec](https://github.com/canonical/snapcraft/blob/main/extensions/desktop/common/mark-and-exec)

The extension then sets this script to run immediately prior to your application using a `command-chain` entry:

```yaml
apps:
  <each application>:
    command-chain:
    - snap/command-chain/desktop-launch
```

### Hooks

The extension sets a [`configure` hook](/t/supported-snap-hooks/3795) to run a script named `hooks-configure-desktop` upon installation, every time the snap is refreshed, and whenever the user changes a configuration option using `snap set` or `snap unset`:

```yaml
hooks:
  configure:
    plugs:
    - desktop
    command-chain:
    - snap/command-chain/hooks-configure-desktop
```

This is a copy of the following script: [`fonts`](https://github.com/canonical/snapcraft/blob/main/extensions/desktop/common/fonts)

### Layout

The extension also sets the following `layout`:

```yaml
layout:
  /usr/share/X11:
    symlink: $SNAP/kf5/usr/share/X11
```
