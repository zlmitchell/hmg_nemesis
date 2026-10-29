# Build and test

How to build NEMESIS for the car and run the test suites. Everything below runs in one
Docker image, so the only host requirements are Docker and KVM.

## 1. Requirements

| | Linux | Windows |
|---|---|---|
| Docker | Docker Engine | Docker Desktop with the WSL2 backend |
| KVM (tests only) | `/dev/kvm` readable by Docker | Run the scripts from a WSL2 shell, where Docker sees `/dev/kvm` |
| Disk | about 11 GB for the image (NDK r21e, SDK, API 19 x86 system image) | same |

The image is `nemesis-api19`, built from [`Sources/tests/api19/Dockerfile`](Sources/tests/api19/Dockerfile).
It contains NDK r21e (as `Sources/native/Makefile` expects), the `android-30` platform jar
(as `Sources/manager/Makefile` expects), build-tools, and a stock Android 4.4 (API 19) x86 emulator. Both
scripts below build the image on first use.

**Line endings.** Device scripts, make recipes and the test scripts must be LF. `.gitattributes`
makes every checkout LF, including on Windows with `core.autocrlf=true`. A clone made before
`.gitattributes` existed can still have CRLF files, so refresh it once from a clean tree:

```sh
git rm -r -q --cached . && git reset -q --hard
```

## 2. Build the car release

```sh
Sources/tools/release.sh <out dir> <keystore dir>
# e.g.  Sources/tools/release.sh ../build ../keys
```

It builds, in the image:

| Output | What |
|---|---|
| `nemesis` | ARM CLI / daemon (`Sources/native`) |
| `NEMESIS.apk` | Manager, signed with `<keystore dir>/nemesis.keystore` |
| `ota_package.tar` | DSP OTA troubleshooter (CLI + Manager + boot hooks), checked by `usb_bundle/verify_tars.sh` |
| `SHA256SUMS` | of the three files |

**Keep the keystore.** It is created on the first run and reused after that. A Manager signed
with a different key will not update over the installed one.

Copy `ota_package.tar` to the root of a FAT32/exFAT stick, then go to Engineering Mode → USB Copy
→ DSP → Update OTA ([03](03-Install-and-Uninstall.md)). Before installing over an existing
upstream install, read [SECURITY-HARDENING.md](SECURITY-HARDENING.md#upgrading-an-existing-install).

### Packages

```sh
python3 Sources/tools/make_nemesis_package.py Sources/packages/<name> -o <name>.zip
```

[04](04-Packages.md) covers the package format.

### Without Docker

Upstream's `Sources/Makefile` (`make`, `make troubleshooter`) works on a Linux ext4 tree with NDK r21e
and the Android SDK. Point it at your copies with `make -C native NDK=…` and
`make -C manager ANDROID_JAR=…`. Some targets reference the original author's paths, for example
`/mnt/c/Users/gmirz/…` and `/home/xwtk/…`.

## 3. Security test suite: stock Android 4.4 emulator

```sh
Sources/tests/api19/run.sh            # the working tree
Sources/tests/api19/run.sh 167b71d    # any git ref, e.g. upstream as a baseline
```

`run.sh` copies `Sources` (or `git archive <ref>`) into a temp dir and builds an x86 CLI and the
Manager (`build.sh`). It then runs `emu_test.sh`, which boots the emulator, installs NEMESIS the
way the troubleshooter does, starts it through the real `install-recovery.sh`, and runs 18
checks. The test script always comes from the working tree, so a baseline run tests *old code
with the current checks*.

| Group | Checks |
|---|---|
| Boot | watchdog started from `install-recovery.sh`; CLI not SUID |
| Root daemon | root for uid 0, 1000, and the `/system/app` Manager; refused for another app uid and for adb shell (uid 2000) |
| Manager | another uid can't start `DialogActivity` or forge the USB prompt reply |
| Files | another app can't write the log; no world-writable dir or file under `/data/nemesis` (including a seeded `0777` `bin/`) |
| USB | a stick inserted while Android is down is held, prompted once Android is up, and runs on Yes; the owner's `usb-recovery` flag allows an unattended run |
| Restore safety | a failing `preuninstall` keeps the package; `install --update` is refused; `wipe` aborts and keeps `/data/nemesis/backup`; uninstall completes once the hook succeeds |

The script prints `PASS`/`FAIL` per check and `passed N, failed M`, and exits non-zero on any
failure. Callers are simulated with `su <uid>`, which gives the daemon the same `SO_PEERCRED` as an
app with that uid.

On this branch the result is 18 passed. Upstream `167b71d` fails 8 of the first 14 checks
([SECURITY-HARDENING.md](SECURITY-HARDENING.md)).

**Not covered:** anything that needs HMG's firmware. That means the DSP OTA install path,
QuickBoot, the OEM signer allowlist, the projection apps, and the Manager's UI on the car.

## 4. Other tests (upstream)

| Test | Needs | What |
|---|---|---|
| `python3 Sources/tests/check_manager_apk.py` | host Python, built APK | Manager APK checks before flashing |
| `python3 Sources/tests/setup_press_test.py` | host JDK | runs `SetupPress` without Android |
| `bash Sources/tests/pkg_live_snapshot.sh` | Linux user + mount namespaces (`unshare`) | `adb.tcp` / `wireless.projection` hooks snapshot and restore the live bytes |
| `bash Sources/tests/pkg_carplay_session.sh` | same | `carplay.session` refuses to replace an APK whose odex doesn't match |
| `Sources/tests/emu_nemesis_flow.sh` | the original author's private Gen5W QEMU image | console-only CLI and USB Handler flow |

The two `pkg_*.sh` scripts hard-code `ROOT=/home/xwtk/nemesis`, so edit that line first. The
Gen5W flow test was updated for this branch but hasn't been run, because its emulator isn't public.

## 5. Before flashing a build

1. `Sources/tests/api19/run.sh` passes.
2. `release.sh` finished with `OK` from `verify_tars.sh`, and `SHA256SUMS` matches the files you copy.
3. The tar on the stick is the one you just built. The prebuilt `ota_package.tar` in the repo root is
   upstream's SUID build.
