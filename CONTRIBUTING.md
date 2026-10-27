# Contributing to malachite-project

Thanks for helping. Fixes and bug reports are welcome from everyone, whatever ROM you
build or use. Reviews can take a while; we read everything.

## Bug reports

Open an issue on the
[device tree](https://github.com/malachite-project/android_device_xiaomi_malachite/issues/new/choose)
and fill in the form. The most important part is the **source ID** of your build:

```
adb shell getprop ro.malachite.source.id
adb shell getprop | grep malachite.source
```

Without it we can't tell which fixes your build has. Also attach a log of the problem
(`adb logcat -b all -d > log.txt` right after it happens) and say how to reproduce it.

## Pull requests

### Before you start

- Base your work on our `lineage-23.2` branch, in a fork made with GitHub's Fork button
  (see [BUILDING.md](BUILDING.md#forking)). Pull requests from a ROM branch carry ROM
  changes with them and can't be merged.
- For a big change, open an issue first so we can agree on the approach.

### Making the change

```
git fetch https://github.com/malachite-project/<repository> lineage-23.2
git checkout -b fix/<short-name> FETCH_HEAD
# make the change, build, test on the phone
git commit
git push <your fork> fix/<short-name>
```

Branch names: `fix/<what>` for fixes, `feature/<what>` for new features.
If the fix was first made on your ROM branch, `git cherry-pick` it onto this branch.

### Commits

Follow LineageOS's style:

```
malachite: keep the charging screen after a power-off

Explain what was wrong and why this fixes it: what you saw, what the
cause was, what the change does. Mention the stock behaviour if the fix
follows it.
```

- Subject: the area (`malachite:`, `sepolicy:`, `audio:`, ...), then what the commit
  does, in under 72 characters, no full stop.
- Body: why. Wrap at 72 columns.
- One logical change per commit, and one topic per pull request.
- Your real name and e-mail as the author. Keep other people's authorship when you use
  their work.
- No unrelated changes: formatting, renames or ROM changes go in their own pull request,
  or not at all.

### Quality

- Build it, and test it on a phone. Say what you tested in the pull request, with the
  source ID of the build you tested.
- Keep stock blobs as they are; changes to `proprietary_vendor_xiaomi_malachite` go
  through `extract-files.py` and `proprietary-files.txt`, never hand-edited files.
- Don't add workarounds that only hide a problem; fix the cause, or explain in the pull
  request why that isn't possible.

### Review

We review every pull request, though it can take some time. We may ask for changes;
push them to the same branch. Accepted pull requests are merged into `lineage-23.2`
after testing, and included in the next release.

## Conduct

Be respectful and assume good intent. Credit other people's work.
