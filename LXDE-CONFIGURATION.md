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
