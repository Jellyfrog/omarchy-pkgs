# Slack on ARM

Slack publishes Linux builds for x86_64 only, and the AUR `slack-desktop` recipe repackages that `.deb`. This package runs the same application code on the AArch64 build of the Electron release Slack ships with, so `yay -S slack-desktop` on an ARM Omarchy install resolves to a working package.

## Electron version

`_electronver` must equal the Electron version inside Slack's `.deb` (`usr/lib/slack/version`). `prepare()` refuses to build when they differ and prints the version to set. A Slack release that moves to a new Electron therefore fails its sync PR build until `_electronver` and the Electron checksum are updated; the upstream watch only tracks Slack's version.

## Native modules

Slack's app carries eight x86_64 Node addons. None can load on AArch64, so they are removed, and the build fails on any other foreign binary so a new native module is reviewed before it ships.

Three are required while Slack boots on Linux and are replaced by JavaScript:

- `file-handler-info` and `electron-native-auth` are MIT-licensed and their Linux builds compile no-op implementations (`impl_none.cc`, `addon_none.cc`). The replacements return the same results.
- `@tinyspeck/slack-desktop-utils` is closed source. Its replacement throws `Method not implemented` from every native call, which is the error Slack's own wrapper uses for methods a platform lacks. Slack catches it, so these Linux features are unavailable: double-press and combo global shortcuts, the huddle screen-share border overlay, audio device detail lookups, clipboard set-and-paste, and haptics.

`@tinyspeck/native-keymap` already catches a missing addon and returns defaults, so keyboard layout detection reports `unknown`. The other four addons are Windows/macOS helpers that Slack does not use on Linux.

## Validation

Installed and used on an aarch64 Omarchy machine (N1x). Features listed above as unavailable were not expected to work and were not checked.
