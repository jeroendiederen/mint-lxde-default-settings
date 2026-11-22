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
