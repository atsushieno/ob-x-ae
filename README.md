# OB-X-AE

## What is this?

OB-X-AE is a modernized builds of [pauloesteban/OB-Xx](https://github.com/pauloesteban/OB-Xx) (which is a fork of [reales/OB-Xd](https://github.com/reales/OB-Xd), namely:

- based on JUCE7/8.
- uses CMake so that we can easily integrate others' JUCE related efforts.
- supports LV2 (JUCE7 feature) and CLAP (with clap-juce-extensions).
- eliminated modal dialogs for Android (for Standalone and AAP plugin).
- automatically install Resources to the user documents directory.

Everything else is left as is, so far.

Everything is still subject to changes. Do not expect it as a stable synth plugin yet.

This repository keeps changes to the dependencies as minimum as possible.
We do not make changes into a fork branch, but has just a `.patch` file to OB-Xx.

### Building on MacOS 15 (Sequoia) or later

JUCE 7.0.12 has a known problem that it uses `CGWindowListCreateImage()` which
is obsoleted and unavailable in macOS 15.0:

>  error: 'CGWindowListCreateImage' is unavailable:
>  obsoleted in macOS 15.0 - Please use ScreenCaptureKit instead.

You will have to either switch to JUCE8, or manually remove uses of this API.

## LICENSES

OB-X-AE is released under the GPLv3 license.

OB-Xx is released under the GPLv3 license.

JUCE (until 7.0.12) is released under the GPLv3 license.

clap-juce-extensions is released under the MIT license.
