# Building ROMs from the malachite trees

This guide is for anyone building LineageOS or another ROM (crDroid, EvolutionX,
RisingOS, ...) for malachite from the malachite-project trees. Following it keeps your
ROM in sync with our fixes, lets your fixes come back to everyone, and makes every build
traceable to its exact source.

**Contents:** [Rules](#rules) · [Which repositories](#which-repositories) ·
[Forking](#forking) · [Setting up your tree](#setting-up-your-tree) ·
[Your ROM's changes](#your-roms-changes) · [Staying in sync](#staying-in-sync) ·
[Building with a source ID](#building-with-a-source-id) · [Releasing](#releasing) ·
[Bug reports](#bug-reports) · [Sending fixes back](#sending-fixes-back)

## Rules

1. **Fork, don't copy.** Use GitHub's Fork button, or push our history to your
   repository. Never re-create a tree by uploading files: that loses the history, and
   then neither our fixes nor yours can be merged.
2. **One branch per ROM, based on ours.** For example `crdroid-16` from `lineage-23.2`.
   Keep our `lineage-23.2` branch in your fork untouched.
3. **Merge our updates, don't re-apply them.** Merge our branch into yours (see
   [Staying in sync](#staying-in-sync)); don't squash, rebase or copy our commits.
4. **Commit and push everything before you build,** patches included. A build from
   uncommitted or unpushed changes can't be traced.
5. **Take a source snapshot before every release build** and put the source ID in your
   release notes.
6. **Keep device fixes and ROM changes in separate commits,** so fixes can be sent back.

## Which repositories

`manifests/malachite.xml` in the device tree lists every repository; the comment at its
top says what each fork changes.

| Kind | Repositories | Use in another ROM |
| --- | --- | --- |
| Device | `android_device_xiaomi_malachite`, `proprietary_vendor_xiaomi_malachite`, `android_device_xiaomi_malachite-kernel` | Fork and use as they are; your ROM changes go on your branch. |
| Hardware | `android_hardware_mediatek`, `android_hardware_xiaomi`, `android_device_mediatek_sepolicy_vndr`, `android_vendor_mediatek_ims`, `android_hardware_dolby` | Use our branch if your ROM tracks LineageOS's; otherwise merge our commits into your ROM's version. |
| Framework | `android_frameworks_base`, `android_frameworks_native`, `android_packages_apps_Settings`, `android_packages_services_Telephony`, `android_packages_modules_Bluetooth`, `android_external_wpa_supplicant_8` | Your ROM has its own forks of these. Cherry-pick our commits onto them (they are on top of LineageOS, by `git log` against LineageOS's branch). |
| Kernel | `kernel_manifest-6.1` and the kernel repositories | Only to build the kernel; the device uses the prebuilt in `malachite-kernel`. |

## Forking

For each repository you need to change:

1. Fork it on GitHub (into your account or your ROM's organisation).
2. In your fork, create your ROM's branch from ours:
   ```
   git clone https://github.com/<you>/android_device_xiaomi_malachite
   cd android_device_xiaomi_malachite
   git checkout -b crdroid-16 origin/lineage-23.2
   git push -u origin crdroid-16
   ```
3. Add ours as a second remote, for updates:
   ```
   git remote add malachite https://github.com/malachite-project/android_device_xiaomi_malachite
   ```

Repositories you don't change: don't fork them, use ours directly.

## Setting up your tree

1. Initialise your ROM's source as its own instructions say (`repo init ...`).
2. Copy our local manifest:
   ```
   mkdir -p .repo/local_manifests
   curl -o .repo/local_manifests/malachite.xml \
     https://raw.githubusercontent.com/malachite-project/android_device_xiaomi_malachite/lineage-23.2/manifests/malachite.xml
   ```
3. Edit it for your ROM:
   - for each repository you forked, add a remote for your account and point the
     project at your fork and branch:
     ```xml
     <remote name="you" fetch="https://github.com/<you>" />
     <project path="device/xiaomi/malachite" name="android_device_xiaomi_malachite"
              remote="you" revision="crdroid-16" />
     ```
   - remove the projects your ROM already provides itself (usually the framework
     forks: frameworks/base, frameworks/native, Settings, Telephony, Bluetooth);
   - keep the rest pointing at malachite-project.
4. Commit that manifest to your device tree fork (for example `manifests/crdroid.xml`),
   so others can rebuild your ROM too.
5. `repo sync`.

## Your ROM's changes

Typical ROM changes: the product makefile (`lineage_malachite.mk` →
`crdroid_malachite.mk`, `AndroidProducts.mk`), ROM flags and overlays.

- Commit them on your ROM branch, separately from fixes. Start the subject with the ROM:
  `malachite: crDroid: add crdroid_malachite product`.
- A fix that isn't specific to your ROM (a bug in the device, a HAL, SELinux) goes in its
  own commit, written so it applies to `lineage-23.2` too, and should be sent back (see
  [Sending fixes back](#sending-fixes-back)).
- Patches your ROM applies with a script must be committed to a fork before building;
  otherwise the build is marked untraceable.

## Staying in sync

When we push fixes, merge them into your ROM branch:

```
cd device/xiaomi/malachite
git fetch malachite lineage-23.2
git merge malachite/lineage-23.2
git push <your remote> HEAD:crdroid-16
```

Do the same for each forked repository. Resolve conflicts in the merge commit; never
rewrite history that you have pushed. Merging rather than copying keeps "which of our
fixes does this build have?" answerable from the commit alone.

## Building with a source ID

Every build made from these trees can be traced back to its exact source.

1. Commit and push all your changes.
2. From the top of the tree, right before building:
   ```
   bash device/xiaomi/malachite/tools/source-snapshot.sh
   ```
   It records the commit of every project in the tree and prints the source ID, for
   example `src-3f9a1c2e0b4d`. If it lists projects with uncommitted or unpushed changes,
   commit and push them and run it again.
3. Build as usual. The record is put in the ROM, so there is nothing extra to publish.

On the phone:

```
adb shell getprop ro.malachite.source.id            # the source ID
adb shell cat /system_ext/etc/source/manifest.xml   # every project and its commit
adb shell getprop | grep malachite.source           # device, vendor and kernel trees
```

These work from adb shell and are included in bug reports. Apps can't read them, so
the source ID doesn't make the ROM easier to detect.

| ID | Meaning |
| --- | --- |
| `src-<12 hex>` | Traceable: every project is at a published commit. |
| `...-untraceable` | Something was uncommitted or unpushed when the snapshot was taken. |
| `...-stale` | The device, vendor or kernel tree changed after the snapshot. Take it again. |
| `none` | No snapshot was taken. |

Anyone can sync the exact tree of a build from its record:
`adb pull /system_ext/etc/source/manifest.xml`, copy it to `.repo/manifests/`, then
`repo init -m manifest.xml` and `repo sync`.

## Releasing

Put these in every release post:

```
ROM:         crDroid 12.x (Android 16)
Source ID:   src-3f9a1c2e0b4d
Device tree: <you>/android_device_xiaomi_malachite@1a2b3c4
Based on:    malachite-project lineage-23.2 @ <commit of our branch you last merged>
```

## Bug reports

Users of every ROM built on these trees can report to us. Ask them to open an issue on
the [device tree](https://github.com/malachite-project/android_device_xiaomi_malachite/issues/new/choose)
with the source ID and logs; the form asks for everything needed. Bugs in your ROM's own
features belong with your ROM.

## Sending fixes back

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: put the fix on a branch made from our
`lineage-23.2` (not from your ROM branch), test it, and open a pull request to our
`lineage-23.2`.
