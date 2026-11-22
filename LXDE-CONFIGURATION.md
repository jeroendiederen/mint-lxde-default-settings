# MintPPC LXDE Configuration

## Purpose

This document describes the configuration of the Mint-LXDE desktop
environment used by MintPPC.

It records the original configuration, the relationships between
desktop components, and the results of configuration experiments.

The goal is to preserve the existing lightweight desktop while
making future maintenance and modernization easier.

## Source baseline

- Package: mint-lxde-default-settings
- Version: 12.0.14
- Git branch: main
- Initial source commit: 301e839

The original package source was imported into Git without modifications.

## Desktop components

| Component | Responsibility |
|-----------|----------------|
| LXDM | Graphical login manager |
| LXSession | Desktop session and GTK-related settings |
| Openbox | Window management and window decorations |
| LXPanel | Desktop panel, menu, taskbar and plugins |
| PCManFM | File manager and desktop management |
| LXAppearance | Graphical interface for appearance settings |

## Configuration locations

The package provides configuration in several locations:

- `etc/skel/` — Default files copied into newly created user homes.
- `etc/xdg/` — System-wide desktop configuration.
- `usr/share/lxpanel/profile/Mint-LXDE/` — LXPanel profile defaults.
- `usr/share/mint-lxde/` — Mint-LXDE resources and configuration.

User-specific settings are stored under `~/.config/`.

The exact configuration precedence of individual components
will be documented as it is verified.

## Configuration experiments

Experiments will record:

1. The original configuration.
2. The setting changed.
3. The observed visual or functional effect.
4. The configuration file modified.
5. The procedure used to restore the original behavior.

Only experimentally verified behavior will be presented as confirmed.

## Open questions

- How do GTK themes affect LXPanel taskbar buttons?
- Which appearance settings are controlled by LXSession?
- How does PCManFM select its desktop appearance?
- What is the precise precedence of system and user configuration?

## Experiment 1: LXPanel background configuration

### Objective

Determine how LXPanel stores its background appearance settings and whether the panel background can be changed independently of the GTK widget theme.

### Environment

- Desktop session: Mint-LXDE
- Panel: LXPanel
- GTK widget theme: Mint-Y
- Configuration file: `~/.config/lxpanel/Mint-LXDE/panels/panel`
- Package baseline: `mint-lxde-default-settings` 12.0.14

### Original configuration

```ini
transparent=0
tintcolor=#000000
alpha=0
background=0
```

The panel used the system theme background.

### Experiment

In LXPanel Panel Preferences, under Appearance, the background was changed from **System theme** to **Solid color (with opacity)**.

The selected color was `#303030`, with full opacity.

The panel immediately appeared dark grey, while the taskbar buttons remained light-colored.

After closing Panel Preferences, the configuration contained:

```ini
transparent=1
tintcolor=#303030
alpha=255
background=0
```

### Restoring the system theme

Selecting **System theme** again and closing Panel Preferences produced:

```ini
transparent=0
tintcolor=#303030
alpha=255
background=0
```

LXPanel retained the previously selected tint and opacity values even though the system theme background was selected.

### Confirmed findings

1. LXPanel can display a solid-color background while the GTK widget theme remains unchanged.
2. Selecting a solid-color background sets `transparent=1` and uses `tintcolor` and `alpha`.
3. The `background` setting remained `0` throughout the experiment.
4. The visual change occurred before the configuration file was updated.
5. Closing Panel Preferences saved the changed settings during this experiment.
6. Returning to the system theme did not reset the stored tint and opacity values.

### Unresolved questions

- Why did the taskbar buttons remain light-colored?
- Which GTK theme rules control their appearance?
- Does LXPanel always save its configuration when Panel Preferences closes, or can other events trigger a save?

### Restoration

The original LXPanel configuration can be restored from:

`etc/skel/.config/lxpanel/Mint-LXDE/panels/panel`

The GTK widget theme must be restored separately through LXAppearance if it was changed during testing.

## Experiment 2: LXPanel taskbar button appearance

### Objective

Determine how LXPanel controls the appearance of taskbar buttons and identify the configuration setting responsible for their raised or flat appearance.

### Environment

- Desktop session: Mint-LXDE
- Panel: LXPanel
- Configuration file: `~/.config/lxpanel/Mint-LXDE/panels/panel`
- Plugin: `taskbar`

### Original configuration

```ini
FlatButton=0
```

The taskbar displayed raised, light-grey buttons.

### Experiment

In Task Bar (Window List) preferences, the **Flat buttons** checkbox was enabled.

The taskbar buttons immediately lost their raised appearance and blended into the panel background.

After closing the preferences window, the configuration contained:

```ini
FlatButton=-1
```

### Restoration

The **Flat buttons** checkbox was disabled again.

After closing preferences, the configuration returned to:

```ini
FlatButton=0
```

### Confirmed findings

1. `FlatButton=0` produces raised taskbar buttons.
2. `FlatButton=-1` produces flat taskbar buttons.
3. The graphical checkbox controls this configuration setting.
4. Closing Task Bar preferences saved the changed value during the experiment.
5. The original appearance was restored successfully.

### Unresolved questions

- How do flat taskbar buttons appear against a dark panel background?
- Does the GTK widget theme influence the colors of raised taskbar buttons?
- Can taskbar button colors be customized independently of the GTK theme?

## Experiment 3: Dark LXPanel and volume icon appearance

### Objective

Create a dark, flat LXPanel while preserving the lightweight Mint-LXDE desktop and ensuring that the volume icons remain clearly visible.

### Environment

- Desktop session: Mint-LXDE
- Panel: LXPanel 0.11.1-3
- Icon theme: Mint-X 1.5.5
- Volume plugin: `volumealsa`
- Test environment: PowerPC Debian QEMU virtual machine
- Package: `mint-lxde-default-settings` 12.0.14

### Original configuration

The original panel used the system theme background, black text and raised taskbar buttons.

```ini
transparent=0
tintcolor=#000000
alpha=0
usefontcolor=1
fontcolor=#000000
background=0
FlatButton=0
```

### Dark panel configuration

The following settings were established through Experiments 1 and 2 and subsequently tested together:

```ini
transparent=1
tintcolor=#303030
alpha=255
usefontcolor=1
fontcolor=#ffffff
background=0
FlatButton=-1
```

The panel displayed a fully opaque dark-grey background with white text and flat taskbar buttons.

The panel height remained 24 pixels, with an icon size of 22 pixels.

### Volume icon problem

The original Mint-X volume icons were dark and difficult to distinguish against the new dark-grey panel.

LXPanel's `volumealsa` plugin requests the following icon names:

- `audio-volume-high-panel`
- `audio-volume-medium-panel`
- `audio-volume-low-panel`
- `audio-volume-muted-panel`

The plugin uses GTK icon-theme lookup functionality through LXPanel.

A newer Mint-X icon package (1.7.5) was inspected, but its corresponding symbolic SVG icons were identical to those in Mint-X 1.5.5.

### Custom MintPPC-Panel icon theme

A small custom icon theme was created to override only the four volume icons.

Theme location in the Debian package:

`usr/share/icons/MintPPC-Panel/`

The `index.theme` file contains:

```ini
[Icon Theme]
Name=MintPPC Panel
Comment=Experimental light panel icons for MintPPC
Inherits=Mint-X
Directories=status/22

[status/22]
Size=22
Context=Status
Type=Fixed
```

Because the theme inherits from Mint-X, icons not supplied by MintPPC-Panel continue to use the inherited icon theme.

### Generating the icons with ImageMagick

The four icons were generated from the existing Mint-X symbolic SVG files.

The original files are located in:

`/usr/share/icons/Mint-X/status/scalable/`

ImageMagick was used to render the SVGs at high resolution and then reduce them to the desired size.

The working procedure was:

```bash
mkdir -p "$HOME/.local/share/icons/MintPPC-Panel/status/22"

for level in high low medium muted; do
    convert -background none -density 384 \
        "/usr/share/icons/Mint-X/status/scalable/audio-volume-${level}-symbolic.svg" \
        -filter Lanczos -resize 18x18 \
        -gravity center -background none -extent 22x22 \
        "$HOME/.local/share/icons/MintPPC-Panel/status/22/audio-volume-${level}-panel.png"
done
```

The important steps are:

1. Render the SVG at 384 DPI to obtain a high-resolution intermediate image.
2. Downsample the image to 18×18 pixels using Lanczos filtering.
3. Center the artwork on a transparent 22×22 pixel canvas.
4. Save the resulting PNG using the icon name expected by LXPanel.

This supersampling technique produced smoother edges than rendering the SVGs directly at their final small size.

The transparent padding allowed the icons to appear comfortably within the 24-pixel-high panel.

The final dimensions were verified with:

```bash
identify -format '%f: %wx%h\n' \
    ~/.local/share/icons/MintPPC-Panel/status/22/*.png
```

All four PNG files measured 22×22 pixels.

### Activating the icon theme

In the PowerPC VM, the LXSession configuration was changed from:

```ini
sNet/IconThemeName=Mint-X
```

to:

```ini
sNet/IconThemeName=MintPPC-Panel
```

The GTK widget theme was left unchanged.

The icon cache was regenerated and LXPanel restarted:

```bash
gtk-update-icon-cache -f ~/.local/share/icons/MintPPC-Panel
lxpanelctl restart
```

The Debian package incorporates the icon-theme setting in:

- `etc/xdg/lxsession/Mint-LXDE/desktop.conf`
- `etc/skel/.config/lxsession/Mint-LXDE/desktop.conf`

### Confirmed findings

1. A dark-grey LXPanel background works with flat taskbar buttons and white panel text.
2. The original Mint-X volume icons were difficult to see against the dark panel.
3. LXPanel's volume plugin uses GTK icon-theme lookup for its volume icons.
4. A custom icon theme can override the four volume icons while inheriting other icons from Mint-X.
5. ImageMagick supersampling produced crisp, light-grey volume icons.
6. The high, medium, low and muted volume states were visually tested successfully in the PowerPC VM.
7. The custom icons displayed correctly within the existing 24-pixel-high panel.

### Package integration

The tested configuration was incorporated into the `mint-lxde-default-settings` source package.

The custom icon theme is installed under:

`/usr/share/icons/MintPPC-Panel/`

The original Mint-X icon files remain untouched.

The four derived PNG icons retain their GPL-3+ licensing and attribution to the Linux Mint project, as documented in `debian/copyright`.

### Restoration

To restore the original panel appearance, use:

```ini
transparent=0
tintcolor=#000000
alpha=0
usefontcolor=1
fontcolor=#000000
background=0
FlatButton=0
```

Restore the original icon-theme setting:

```ini
sNet/IconThemeName=Mint-X
```

Restart LXPanel or log out and back in to apply the restored configuration.

### Remaining verification

The individual settings and volume icons have been tested in the PowerPC VM.

The complete updated Debian package still requires an installation test to verify that its system-wide icon theme and default LXPanel configuration work correctly together.
