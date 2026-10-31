# malachite-project

Unofficial LineageOS, crDroid and Evolution X for the Redmi Note 14 Pro 5G / POCO X7 5G
(malachite), and the device trees that other ROMs for malachite can be built on.

**Downloads, updates and support:** [Telegram group](https://t.me/malachite_project)

## Branches

| Branch | ROM | Android | Status |
| --- | --- | --- | --- |
| `lineage-24.0` | LineageOS 24.0 | 17 | Main line. Base your work and your ROMs on this. |
| `crdroid-17.0` | crDroid 13.0 | 17 | Kept level with `lineage-24.0`. |
| `evolutionx-24.0` | Evolution X 12.3 | 17 | Kept level with `lineage-24.0`. |

`lineage-23.2` (Android 16) is retired. Every repository uses the same branch names;
the crDroid and Evolution X branches exist where those ROMs need a change. Start from the
[device tree](https://github.com/malachite-project/android_device_xiaomi_malachite):
its `manifests/malachite.xml` lists every repository and what each one changes.

## Building a ROM from these trees

Read [BUILDING.md](https://github.com/malachite-project/android_device_xiaomi_malachite/blob/lineage-24.0/BUILDING.md):
how to fork, keep your ROM in sync with our fixes, and give every build a source ID that
traces it to the exact commits it was made from.

## Bugs and contributions

- **Bug reports** for any ROM built on these trees:
  [device tree issues](https://github.com/malachite-project/android_device_xiaomi_malachite/issues/new/choose).
  Include your build's source ID.
- **Fixes:** pull requests are welcome; see
  [CONTRIBUTING.md](https://github.com/malachite-project/.github/blob/main/CONTRIBUTING.md).

## Credits

Built on the work of [mt6878-devs](https://github.com/mt6878-devs),
[puhboo](https://github.com/puhboo) and [cmdr-chara](https://github.com/cmdr-chara),
and of the LineageOS project.
