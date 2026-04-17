# bellum-linux
Get Bellum working on proton with Linux 


# Prereqs
- Lutris installed
- GE-Proton10-34 installed



# bellum-linux

Get Bellum (milsim shooter) working on Linux with Lutris

# Prereqs

* Lutris installed
* GE-Proton installed (tested with GE-Proton10-34) — this goes in `~/.local/share/Steam/compatibilitytools.d/` or Lutris's equivalent runner directory
* An X11 session — **Wayland does not work** (confirmed broken on KDE Plasma 6 Wayland, the launcher's WebView2 renders as a white screen). Use i3, X11 Plasma, Xfce, or any X11 session. Check with `echo $XDG_SESSION_TYPE` — it should say `x11` not `wayland`
* A Windows machine or VM for one-time email verification (explained below, hopefully this can be removed in the future)

# The weird part up front

This setup requires swapping runners mid-install because of how the Astarte Launcher behaves:

1. The **installer** (`AstarteLauncher-amd64-installer.exe`) is itself a WebView2 app. Under GE-Proton it renders but hits issues; under plain Wine it runs invisibly but completes the file extraction successfully. So we use **plain Wine to install**.
2. The **installed launcher** (`AstarteLauncher.exe`) needs GE-Proton's bundled patches (media foundation, DXVK, etc.) to render WebView2 properly. Plain Wine gives a white screen. So we use **GE-Proton to run**.

Yes it's annoying. Yes I'll probably turn this into a Lutris install script eventually.

# Setup

* Download the Astarte Launcher installer from the Bellum website (`AstarteLauncher-amd64-installer.exe`) [here](https://playbellum.com/download/)
* Download the WebView2 Evergreen runtime installer from [Microsoft](https://msedge.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/92da24d5-13a1-4d0c-8592-b6056b909dd4/MicrosoftEdgeWebView2RuntimeInstallerX64.exe)
* In Lutris, create a new game entry pointing at the Astarte installer — I used a dedicated prefix at `/home/<user>/Games/bellum` but anywhere will do
* Configure the Lutris runner to be **plain Wine** (I used `wine-10.20-amd64`) for now — we'll swap to GE-Proton later
* Before running the installer, install WebView2 into the prefix. Open a Wine shell against your prefix and run the WebView2 installer, or use Lutris's "Run EXE inside Wine prefix" option to run `MicrosoftEdgeWebView2Setup.exe`. Let it finish — it's silent but it works
* Now run the Astarte installer. It will appear to do nothing — no window, no progress bar, just silence. This is expected. Wait ~30 seconds, then check `<prefix>/drive_c/users/steamuser/AppData/Local/Astarte Industries/Astarte Launcher/` — you should see `AstarteLauncher.exe` there. If you do, the silent install worked
* If the install directory doesn't exist, the installer actually failed — check `ps aux | grep -i astarte` to see if it's still running, and give it more time
* Once the launcher is installed, **switch the Lutris runner to GE-Proton10-34** (Configure → Runner options → Wine version)
* Update the Lutris game's executable path to point at the installed launcher instead of the installer:
  `<prefix>/drive_c/users/steamuser/AppData/Local/Astarte Industries/Astarte Launcher/AstarteLauncher.exe`
* Remove any `--no-sandbox --disable-gpu` arguments from the Lutris launch arguments — they aren't needed and didn't help anyway
* Launch the game from Lutris. The launcher should now open properly and show the Astarte UI

# Account setup (the Windows VM part)

Astarte uses a custom URI scheme (`astarte://`) to handle email verification during account creation. When you click the verification link in your email, it tries to hand off to the launcher via this protocol handler, which isn't registered on Linux by default.

I haven't figured out the full protocol handler registration yet (similar to what I did for [pokemon-tcg-live-steam](https://github.com/catdevman/pokemon-tcg-live-steam) with `xdg-mime` and Wine registry edits). As far as I can tell the verification handoff only happens **once** during initial account setup, so the workaround is:

* Spin up a Windows VM (or borrow a Windows machine)
* Install the Astarte Launcher on Windows
* Create your account and complete email verification there
* Log out of the Windows install
* Log in on your Linux install — it should work without needing the protocol handler since verification is already done

If anyone discovers the verification flow fires again later (password reset, re-auth, multi-device login), open an issue here and I'll work on the proper protocol handler solution.

# Redeeming your key

Once you're logged in on Linux, key redemption happens through the launcher UI normally. No additional workarounds needed as far as I can tell.

# Known Issues

* **Wayland**: Launcher renders as a white screen under Wayland sessions. Confirmed broken on KDE Plasma 6 Wayland. Use an X11 session. This may be fixable in the future as Wine's Wayland driver matures
* **Silent installer**: The installer runs without any visible UI under plain Wine. It's working, just invisible
* **WebView2 version sensitivity**: Evergreen WebView2 works with GE-Proton10-34. If Microsoft pushes a breaking Evergreen update, you may need to fall back to a [Fixed Version](https://developer.microsoft.com/en-us/microsoft-edge/webview2/) install — note that Microsoft only exposes the two most recent major versions in their dropdown now
* **Custom URI scheme**: `astarte://` protocol handler is not registered on Linux, hence the Windows VM workaround for email verification

# What didn't work (so you don't waste time)

* Plain Wine for running the installed launcher → white screen
* GE-Proton under Wayland → white screen
* `WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS` with various GPU-disabling flags → no effect on the white screen
* Installing Fixed Version WebView2 (146.0.3856.109) → didn't fix the underlying Wayland issue, unnecessary once on X11 + GE-Proton
* Passing `--no-sandbox --disable-gpu` as launcher arguments → these flags don't propagate to WebView2's internal Chromium

# Tested with

* Arch Linux (KDE Plasma 6, i3)
* GE-Proton10-34
* Wine 10.20-amd64 (for installer only)
* WebView2 Evergreen runtime (current as of April 2026)
* Lutris (current stable)

# TODO

* Write a proper Lutris install script so this becomes one-click
* Figure out the `astarte://` protocol handler registration to remove the Windows VM requirement
* Test on NVIDIA (I'm on AMD via radv)
* Test on Gnome X11

Note: This writeup was produced with assistance from Claude (Anthropic). Steps were tested on my setup but please verify commands and paths before running them on yours.
