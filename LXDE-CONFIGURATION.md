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

## Experiment 4: Modern NetworkManager Ethernet icons

### Objective

Replace the old NetworkManager Ethernet icons with modern, light-grey Mint-X symbolic icons that match the dark LXPanel and the custom volume icons introduced in Experiment 3.

Maintain compatibility with the existing `nm-applet` implementation without modifying NetworkManager or the original system icon files.

### Environment

- PowerPC Debian virtual machine running MintPPC LXDE.
- LXPanel with a 24-pixel-high dark-grey panel.
- NetworkManager tray application (`nm-applet`).
- Custom GTK icon theme: `MintPPC-Panel`, inheriting from `Mint-X`.
- ImageMagick for SVG-to-PNG conversion.

### Investigation

The LXPanel configuration contains a `tray` plugin but no dedicated network-status plugin.

The running `nm-applet` process was confirmed using:

```bash
pgrep -a nm-applet
```

The installed `nm-applet` binary references the legacy icon names `nm-device-wired` and `nm-no-connection`.

The original `nm-device-wired` icon is provided by the `hicolor` icon theme and uses an older GNOME-style design.

Mint-X provides modern symbolic alternatives:

- `network-wired-symbolic.svg`
- `network-wired-disconnected-symbolic.svg`

These icons use light-grey shapes (`#bebebe` in the connected SVG) suitable for a dark panel.

### Icon conversion

The modern Mint-X symbolic icons were converted to PNG files using ImageMagick, following the same approach as the volume icons in Experiment 3.

The images were resized to 18×18 pixels and centered on transparent 22×22 canvases.

**Connected Ethernet:**

```bash
convert -background none -density 384 \
    /usr/share/icons/Mint-X/status/scalable/network-wired-symbolic.svg \
    -filter Lanczos -resize 18x18 \
    -gravity center -background none -extent 22x22 \
    /tmp/nm-device-wired.png
```

**Disconnected Ethernet:**

```bash
convert -background none -density 384 \
    /usr/share/icons/Mint-X/status/scalable/network-wired-disconnected-symbolic.svg \
    -filter Lanczos -resize 18x18 \
    -gravity center -background none -extent 22x22 \
    ~/.local/share/icons/MintPPC-Panel/apps/22/nm-no-connection.png
```

The connected icon was copied into the same custom icon directory.

### Custom icon theme

The following files were added to the `MintPPC-Panel` theme:

```text
apps/22/nm-device-wired.png
apps/22/nm-no-connection.png
```

The theme's `index.theme` file was extended:

```ini
[Icon Theme]
Name=MintPPC Panel
Comment=Experimental light panel icons for MintPPC
Inherits=Mint-X
Directories=status/22,apps/22

[status/22]
Size=22
Context=Status
Type=Fixed

[apps/22]
Size=22
Context=Applications
Type=Fixed
```

The existing volume icons remain in `status/22`.

The legacy NetworkManager filenames are retained so that `nm-applet` can locate the replacements through GTK's icon-theme mechanism.

### Testing

The icon theme was refreshed and `nm-applet` restarted in the PowerPC VM.

The connected Ethernet icon changed successfully to the modern, light-grey design.

The new icon is visually consistent with the dark panel and the custom volume icons.

The disconnected Ethernet icon was created and its file presence verified, but its appearance during an actual network disconnection has not yet been tested.

### Repository integration

Both Ethernet PNG files were copied from the PowerPC VM to the ZenBook source repository:

```text
usr/share/icons/MintPPC-Panel/apps/22/nm-device-wired.png
usr/share/icons/MintPPC-Panel/apps/22/nm-no-connection.png
```

The repository's `index.theme` was updated to register `apps/22`.

The original NetworkManager and Mint-X icon files remain unchanged.

These PNG files are derived from Mint-X artwork. Their GPL-3+ licensing and attribution should be included in `debian/copyright` before committing.

### Remaining verification

1. Test the disconnected Ethernet icon when a safe opportunity arises.
2. Verify that the icons work when installed system-wide by the Debian package.
3. Verify icon-theme inheritance and application compatibility after installation.
4. Complete the Debian package integration test together with the changes from Experiment 3.

No Debian package build was performed during this experiment.

## Experiment 5: Update indicator icons and clipboard manager exclusion

### Objective

Modernize the package update indicator icons to match the dark MintPPC LXDE panel.

Replace the old, visually inconsistent update icons with light-grey artwork that remains clearly visible against the dark-grey panel.

Additionally, remove unwanted clipboard managers Diodon and CopyQ from the default MintPPC installation without disabling other LXDE recommendations.

### Environment

- Desktop session: Mint-LXDE
- Panel: LXPanel, 24 pixels high
- Icon theme: MintPPC-Panel
- Parent icon theme: Mint-X
- Test environment: PowerPC Debian QEMU virtual machine
- Package: `mint-lxde-default-settings`

### Update indicator icons

Two custom icons were added to the MintPPC-Panel icon theme:

| Icon file | Purpose |
|---|---|
| `status/22/software-update-available.png` | Updates available (grey star) |
| `apps/22/system-software-update.png` | Update indicator (grey shield) |

The grey star was rendered from the Mint-X symbolic update-available artwork.

The shield was derived from:

`/usr/share/icons/Mint-Y/status/symbolic/security-low-symbolic.svg`

The original Mint-Y icon is provided by `mint-y-icons`.

### Icon generation

Both icons use the same ImageMagick rendering procedure established in Experiment 3:

1. Render the original SVG at 384 DPI.
2. Resize the artwork to 18×18 pixels using Lanczos filtering.
3. Center it on a transparent 22×22 pixel canvas.
4. Save the PNG under the filename expected by the update indicator.

The resulting files were verified as 22×22 pixel PNG images with grey artwork and alpha transparency.

The shield conversion command was:

```bash
convert -background none -density 384 \
    /usr/share/icons/Mint-Y/status/symbolic/security-low-symbolic.svg \
    -filter Lanczos -resize 18x18 \
    -gravity center -background none -extent 22x22 \
    ~/.local/share/icons/MintPPC-Panel/apps/22/system-software-update.png
```

### Mint-X update-available icon

The grey star was generated from the Mint-X symbolic update-available artwork:

`/usr/share/icons/Mint-X/status/scalable/software-update-available-symbolic.svg`

The conversion procedure was:

```bash
convert -background none -density 384 \
    /usr/share/icons/Mint-X/status/scalable/software-update-available-symbolic.svg \
    -filter Lanczos -resize 18x18 \
    -gravity center -background none -extent 22x22 \
    ~/.local/share/icons/MintPPC-Panel/status/22/software-update-available.png
```

### Clipboard manager investigation

The orange Diodon panel icon was traced to the Diodon clipboard manager.

The dependency chain was investigated using:

```bash
aptitude why diodon
```

The result was:

```text
lxde Recommends diodon | copyq
```

Neither Diodon nor CopyQ is wanted in the default MintPPC installation.

The LXDE package recommends them as alternatives, rather than requiring them as hard dependencies.

### APT preferences

To prevent either clipboard manager from being installed through normal APT dependency resolution, the following file was added:

`etc/apt/preferences.d/mintppc-no-clipboard.pref`

Contents:

```text
Package: diodon copyq
Pin: version *
Pin-Priority: -1
```

This excludes both packages without disabling other LXDE recommendations.

### Planned installer cleanup

As an additional precaution, the Debian Installer preseed configuration will purge both packages using a late command:

```bash
in-target apt-get -y purge diodon copyq
```

This installer change is still pending integration and testing.

The APT preferences file and installer purge serve complementary purposes: the purge removes packages selected during installation, while the negative APT priority prevents their subsequent installation through normal APT dependency resolution.

### Testing and verification

The following work has been completed:

1. Both update icons were generated in the PowerPC VM.
2. Their appearance was inspected in the running LXDE desktop.
3. Both PNG files were copied to the ZenBook Git repository.
4. Their dimensions were verified as 22×22 pixels.
5. Their copyright and licensing information was added to `debian/copyright`.
6. The APT preferences file was created.

### Remaining work

1. Complete the Debian package build.
2. Install and test the package in the PowerPC VM.
3. Verify both update icons after a fresh login.
4. Verify that neither Diodon nor CopyQ is installed.
5. Integrate the purge command into the installer preseed configuration.
6. Test a fresh installation and confirm that other LXDE recommendations remain unaffected.

## MintPPC GTK theme configuration

### Desktop theme

The default GTK theme is Mint-X.

The icon theme is MintPPC-Panel, which inherits from Mint-L.
This provides modern Mint category and application icons while
preserving the custom MintPPC panel status icons.

### LXPanel dark menu

LXPanel uses a separate GTK theme, MintPPC-Panel-Dark.

This theme inherits from Mint-L-Dark and overrides the menu
background colour to #303030.

LXPanel is started with:

    lxpanel --profile Mint-LXDE --gtk-theme MintPPC-Panel-Dark

The --gtk-theme option is a MintPPC addition to LXPanel 0.11.1.
It applies the GTK theme through GtkSettings within the LXPanel
process.

Unlike the GTK_THEME environment variable, this does not affect
applications launched from the panel. They retain the normal
desktop GTK theme.

### Panel branding

The LXPanel menu uses:

    linuxmint-logo-ring-symbolic.svg

### Desktop shortcuts

The default Home and Computer desktop shortcuts are removed.
Users can still access their home directory and filesystem
through PCManFM.

## Implementation status — 26 November 2026

This section supersedes the historical pending tasks in the
previous experiments where applicable.

### LXPanel package dependency

The mint-lxde-default-settings package requires:

    lxpanel (>= 0.11.1-3+mintppc1)

This version introduces the process-local --gtk-theme option
required by the Mint-LXDE session.

The mintsystem package provides APT pinning through:

    /etc/apt/preferences.d/99-mintppc.pref

LXPanel packages originating from mintppc.nl receive priority
1001, protecting the customized packages against replacement
by unpatched Debian packages.

### System tray background refresh

A problem was discovered where embedded system tray icons
acquired black backgrounds after dismissing XScreenSaver.

Restarting LXPanel or changing the panel height restored the
correct backgrounds. Running xrefresh did not resolve the issue.

Monitoring the panel window with xev confirmed that visibility
notification events were received during screensaver activation
and dismissal.

A patch was developed to refresh panel and embedded system tray
backgrounds when the panel becomes visible again after being
fully obscured.

The patch reuses the existing _panel_queue_update_background()
function and introduces no polling or additional dependencies.

The fix is included in LXPanel 0.11.1-3+mintppc2.

Successful runtime testing was performed on MintPPC32 in QEMU
with both animated and blank screensaver modes.

The package was also successfully built for ppc64.
Runtime testing on ppc64 and physical hardware remains pending.

### Upstream contributions

Both LXPanel improvements were submitted upstream as independent
pull requests on 26 November 2026:

- https://github.com/lxde/lxpanel/pull/126
  Add optional --gtk-theme command-line argument.

- https://github.com/lxde/lxpanel/pull/127
  Refresh panel backgrounds when becoming visible again.

Neither contribution contains MintPPC-specific theme names or
configuration requirements.

### Installer integration

The MintPPC32 and MintPPC64 preseed configurations now include
the removal of Diodon and CopyQ.

The installer also removes the existing default-display-manager
file before selecting and reconfiguring LXDM.

This prevents the previous display manager selection from
overriding the intended LXDM configuration.

### Installation verification

A fresh MintPPC32 installation in QEMU successfully reached
the Mint-LXDE desktop using LXDM.

The modernized panel, dark menu, light application theme and
custom system tray icons were verified.

The system tray background refresh fix was subsequently
installed and tested successfully.

A fresh MintPPC64 installation is currently in progress.

### Remaining work

- Complete fresh MintPPC64 installation verification.
- Verify APT pinning on both fresh installations.
- Test the modernized desktop on physical PowerPC hardware.
- Test system tray background refresh on physical hardware.
- Remove the nonfunctional native LXPanel batt plugin from
  both default panel configurations after reviewing hardware
  compatibility.
- Inspect and modernize mac-battery-applet to match the new
  panel colours while preserving battery detection functionality.
- Test mac-battery-applet on the PowerBook G4.
- Follow the two upstream LXPanel pull requests.

The experimental BackgroundColor change to the native batt
plugin was reverted. Battery applet modernization is deferred
until physical hardware testing.
