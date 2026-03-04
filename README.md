# 🥽 ALVR on Debian Linux

> **Stream PC VR games wirelessly to your standalone headset via SteamVR on Linux.**  
> Covers both **Native Steam** and **Flatpak Steam** installs.

---

## 📋 Step 0 — Check Your Setup First

Before doing anything, answer these questions. Your answers decide which path to follow.

### How is Steam installed?

Run this in a terminal:

```bash
flatpak list | grep Steam
```

| Result | Means |
|--------|-------|
| You see output with Steam | → Follow **Part B (Flatpak)** |
| Nothing shows up | → Follow **Part A (Native)** |

---

### Is SteamVR installed?

Open Steam → Library → search **SteamVR**.  
If it's not installed: install it, **launch it once, then close it**. This is required before ALVR will work.

---

### What GPU do you have?

```bash
lspci | grep -i vga
```

Note whether it says **AMD**, **Intel**, or **NVIDIA**. This matters if you're on a laptop with two GPUs — see the [Laptop section](#-laptop-two-gpus-read-this) below.

---

### Other things you'll need

- ✅ Your PC and headset on the **same Wi-Fi network** (5 GHz strongly recommended)
- ✅ A **USB cable** — needed once to install the ALVR app on your headset
- ✅ A compatible headset: Meta Quest 1/2/3/Pro, Pico 4, or similar Android-based standalone headset

---

## Part A — Native Steam

> Follow this if Steam was installed via `apt` or Valve's `.deb` package.

### A1 — Download the ALVR Launcher

```bash
cd ~/Downloads
wget https://github.com/alvr-org/ALVR/releases/latest/download/alvr_launcher_linux.tar.gz
tar -xzf alvr_launcher_linux.tar.gz
mkdir -p ~/alvr && mv alvr_launcher_linux/* ~/alvr/
```

**✅ Expected output:** No errors. Confirm with `ls ~/alvr` — you should see files there.

---

### A2 — Run the Launcher

```bash
~/alvr/"ALVR Launcher"
```

If you get a permission denied error:

```bash
chmod +x ~/alvr/"ALVR Launcher"
```

**✅ Expected output:** The ALVR Launcher window opens with an empty version list.

---

### A3 — Install the Streamer

1. Click **Add version**
2. Leave channel and version on their defaults
3. Click **Install** and wait for the download to finish

**✅ Expected output:** A version entry appears in the list with a **Launch** button next to it.

---

### A4 — Install the ALVR App on Your Headset

You have three options — pick the easiest one for you:

#### Option 1: Meta Quest App Store (easiest) ⭐
Install directly from the Meta Quest store — no cables or faffing around needed:
- **On your phone:** Open the Meta Quest app → Store → search **ALVR** → Install
- **In the headset:** Open the store, search **ALVR** → Install

#### Option 2: ALVR Launcher (APK install)
Connect your headset via USB. Your headset may show a popup asking to **allow USB debugging** — tap Allow.

In the ALVR Launcher, click **Install APK**.

#### Option 3: Manual adb install
If neither of the above work:

```bash
sudo apt install adb
adb devices
```

**✅ Expected output from `adb devices`:**
```
List of devices attached
1WMHH8XXXXXXXX    device
```
If it says `unauthorized`, check your headset for the permission popup. Then install the APK:

```bash
adb install ~/alvr/data/alvr_client_android.apk
```

---

### A5 — First Launch and Pairing

1. In the ALVR Launcher, click **Launch**
2. A setup wizard opens — follow it (it sets up firewall rules automatically)
3. Put on your headset and open the **ALVR app** from your headset's app library
4. Back on your PC, in the ALVR Dashboard → **Devices** tab — your headset should appear
5. Click **Trust** next to it
6. The stream starts 🎉

**✅ Expected output:** Your headset shows the SteamVR home environment.

---

## Part B — Flatpak Steam

> Follow this if Steam was installed via Flatpak from Flathub.  
> ⚠️ Flatpak support is experimental but works. Do **Steps A1–A4** first, then come back here.

---

### B1 — Fix SteamVR Permissions

Flatpak's sandbox prevents SteamVR from setting a required permission itself. Run this **once** after installing SteamVR:

```bash
sudo setcap CAP_SYS_NICE+eip ~/.var/app/com.valvesoftware.Steam/data/Steam/steamapps/common/SteamVR/bin/linux64/vrcompositor-launcher
```

Then launch SteamVR once and close it.

**✅ Expected output:** No output at all = success. An error about the file path means SteamVR isn't installed yet.

---

### B2 — Set SteamVR Launch Options

In Steam: right-click **SteamVR** → **Properties** → **General** → **Launch Options**.

Paste this:

```
~/.var/app/com.valvesoftware.Steam/.local/share/Steam/steamapps/common/SteamVR/bin/vrmonitor.sh %command%
```

---

### B3 — Fix Audio (pipewire)

If you get pipewire errors when launching, run:

```bash
flatpak override --user --filesystem="xdg-run/pipewire-0" com.valvesoftware.Steam
```

Then restart Steam and SteamVR.

**✅ Expected output:** No output = success.

---

### B4 — Launch ALVR

Due to the Flatpak sandbox, launch ALVR with:

```bash
flatpak run --command=alvr_launcher com.valvesoftware.Steam
```

Then follow [Step A5](#a5--first-launch-and-pairing) to pair your headset.

---

### Optional: Desktop Shortcut

So you don't have to type that command every time:

```bash
cp ~/alvr/alvr/xtask/flatpak/com.valvesoftware.Steam.Utility.alvr.desktop \
   ~/.local/share/flatpak/exports/share/applications/
```

Log out and back in — ALVR should appear in your app launcher.

---

## 💻 Laptop? Two GPUs? Read This

Laptops often have two GPUs: a weak integrated one and a powerful dedicated one. SteamVR needs the powerful one. If things are slow or broken, add one of these to SteamVR's **Launch Options**:

### AMD/Intel integrated + AMD/Intel dedicated

```
DRI_PRIME=1 [your existing launch options]
```

### AMD/Intel integrated + NVIDIA dedicated

```
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia [your existing launch options]
```

<details>
<summary>Full Flatpak + NVIDIA example (click to expand)</summary>

```
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia QT_QPA_PLATFORM=xcb ~/.var/app/com.valvesoftware.Steam/.local/share/Steam/steamapps/common/SteamVR/bin/vrmonitor.sh %command%
```

</details>

---

## 🔧 Troubleshooting

| Problem | Fix |
|---------|-----|
| Headset not showing up in Devices tab | Make sure PC and headset are on the same Wi-Fi. Try 5 GHz. Check firewall isn't blocking ALVR. |
| `adb devices` shows nothing | Check headset is plugged in. Look for a USB permission popup on the headset. Try a different cable. |
| `adb devices` says `unauthorized` | Check your headset — there should be a popup asking to allow USB debugging. |
| SteamVR won't start | Launch SteamVR on its own at least once first, then close it before using ALVR. |
| Laggy / stuttery stream | Use 5 GHz Wi-Fi. Move closer to your router. Or try a [wired USB connection](https://github.com/alvr-org/ALVR/wiki/ALVR-wired-setup-(ALVR-over-USB)). |
| Flatpak: pipewire errors | Run the `flatpak override` command from Step B3 and restart. |
| Flatpak: new Steam window opens on launch | Close both Steam and ALVR. Open Steam first, wait for the store to load, then launch ALVR. |

---

## 🔗 Useful Links

- [ALVR Releases (download page)](https://github.com/alvr-org/ALVR/releases/latest)
- [Linux Troubleshooting Wiki](https://github.com/alvr-org/ALVR/wiki/Linux-Troubleshooting)
- [Wired USB Setup Guide](https://github.com/alvr-org/ALVR/wiki/ALVR-wired-setup-(ALVR-over-USB))
- [Settings Tutorial](https://github.com/alvr-org/ALVR/wiki/Settings-tutorial)
