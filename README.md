# Linux macOS Recovery USB Creator [![Stars](https://img.shields.io/github/stars/im0x/Linux-macOS-Recovery-USB-Creator?style=social)](https://github.com/im0x/Linux-macOS-Recovery-USB-Creator)

Create a bootable macOS Recovery USB from any major Linux distro (Debian/Ubuntu, Fedora, Arch, etc.). Targets Monterey (12.6) by default; adapts for Ventura/Sonoma via fetch-macOS.py. Uses Apple's BaseSystem.dmg for legal, direct recovery boot—no full installer bloat.

Tested Nov 2025 on Ubuntu 25.10 (sudo-rs default), Fedora 42, Arch rolling. Outputs ~3GB raw IMG; dd to USB. Bypasses Internet Recovery flakes (e.g., stuck on Sierra).

## Prereqs
- Git, Python 3, wget/curl.
- USB ≥16GB (gets wiped).
- ~20GB disk space.
- Stable net (downloads ~3GB).
- Root access (sudo/sudo-rs; no diff in usage).

## Install Deps
Run the block for your distro. For xar: Use pkg mgr if available (e.g., apt on Ubuntu); fallback to source build.

| Distro       | Command |
|--------------|---------|
| Debian/Ubuntu | `sudo apt update && sudo apt install -y git python3 wget hfsprogs dmg2img build-essential autoconf libxml2-dev libssl-dev mac-fdisk-cross xar` |
| Fedora       | `sudo dnf install -y git python3 wget hfsprogs dmg2img @development-tools libxml2-devel openssl-devel` |
| Arch         | `sudo pacman -Syu --needed git python wget hfsprogs dmg2img base-devel libxml2 openssl` |

If xar not in repos (Fedora/Arch), build from source (fixes OpenSSL in newer libssl):
```bash
cd ~
git clone https://github.com/mackyle/xar
cd xar/xar  # Note: subdir has the goods
sed -i 's/OpenSSL_add_all_ciphers/OPENSSL_init_crypto/g' configure.ac
./autogen.sh --prefix=/usr/local
make
sudo make install  # Or sudo-rs if on Ubuntu 25.10+
cd ~  # xar --version to verify
```

## Fetch & Prep BaseSystem
Clone fetch-macOS (KVM helpers; MIT-licensed).
```bash
git clone --depth 1 --recursive https://github.com/kholia/OSX-KVM.git
cd OSX-KVM
./fetch-macOS-v2.py  # Menu: 5 for Monterey (12.6); 6 Ventura, 7 Sonoma
cp BaseSystem.dmg ~/recovery/  # Or your dir
cd ~/recovery
dmg2img BaseSystem.dmg BaseSystem.img  # Expands to ~3GB raw
```

## Burn to USB
ID your USB (whole device, not partition):
```bash
lsblk -o NAME,SIZE,TYPE  # e.g., sdb 16G disk
```
Burn (replace `/dev/sdX`; ~5min):
```bash
sudo dd if=BaseSystem.img of=/dev/sdX bs=4M status=progress && sync
sudo eject /dev/sdX  # Or udisksctl power-off -b /dev/sdX
```

## Boot & Recover
- Insert USB on Mac.
- Restart + hold Option (⌥).
- Select EFI Boot/Recovery (~3GB volume).
- In Recovery: Disk Utility → new APFS vol (GUID scheme).
- Reinstall macOS → downloads fresh (needs net).

## Troubleshooting
| Issue | Fix |
|-------|-----|
| xar sed fails | Ensure in `xar/xar`; re-clone if git hiccup. |
| fetch.py hangs | VPN/proxy; retry option. For Sequoia (15), use 8—needs signed EFI (may fail on old hardware). |
| dd "No space" | Use ≥16GB USB; verify lsblk. |
| No boot menu | Reset Mac NVRAM (Cmd+Opt+P+R at chime); try USB 3.0 port. Alt: resize BaseSystem p2 with parted if needed (rare). |
| sudo-rs weird logs | Ubuntu 25.10+ default (Rust impl); ignore—behavior identical to sudo. |

## Notes
- For full installer: Extend with macos_usb repo (github.com/notthebee/macos_usb); but Recovery suffices for wipes/reinstalls.
- Legal: Pulls from swscan.apple.com—Apple's public mirrors.

## Credits
- [OSX-KVM](https://github.com/kholia/OSX-KVM) for fetch.py.
- [coolaj86 Gist](https://gist.github.com/coolaj86/9834a45a6c21a41e8882698a00b55787) for BaseSystem direct-dd (core hack).
- Tested: MacBook Air 2015 → Monterey clean install.
