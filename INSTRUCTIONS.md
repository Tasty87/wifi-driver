# Build your USB Wi-Fi driver in the cloud (phone only)

Your TP-Link TL-WN823N (RTL8192EU) on a `creek`/`khaje` phone, kernel
`5.15.194-android13-8-...-ab15212794`. No PC needed — GitHub builds it.

The build makes TWO things:
1. **`rtl8xxxu.ko` + `mac80211.ko`** — mainline in-tree driver. **Try this first**, it's
   the one most likely to load. Good for normal Wi-Fi (connect to networks).
2. **`8192eu.ko`** — vendor driver, adds monitor mode / injection. Bonus; may fail to build.

---

## Step 1 — Make a GitHub account (phone browser)
- Go to https://github.com/signup — free. Verify your email.

## Step 2 — Create an empty PUBLIC repo
- Tap **+** (top right) → **New repository**.
- Name: `wifi-driver` (anything). Visibility: **Public** (public = free unlimited build minutes).
- Tick **Add a README file** so the repo isn't empty. **Create repository**.

## Step 3 — Add the workflow file
- In the repo, tap **Add file** → **Create new file**.
- In the filename box type exactly:  `.github/workflows/build.yml`
  (typing the `/` characters creates the folders automatically)
- Paste the entire contents of `build.yml` (the file I give you) into the editor.
- Scroll down → **Commit changes**.

## Step 4 — Run it
- Tap the **Actions** tab.
- If it says workflows are disabled, tap the green **I understand… enable** button.
- Pick **build-wifi-driver** on the left → **Run workflow** → **Run workflow**.
- It runs on GitHub's servers for roughly **40–70 minutes**. You can close the browser;
  come back to the Actions tab to check. A green ✓ means success.

## Step 5 — Download the modules to your phone
- Open the finished run → scroll to **Artifacts** → tap **wifi-modules** to download the zip.
- Unzip it (any file manager). You'll get `rtl8xxxu.ko`, `mac80211.ko`, maybe `8192eu.ko`.

## Step 6 — Tell me when you have the files
- Come back here, say you've got them and where they are on the phone.
- I'll walk you through loading them as **root** (with the dongle plugged in):
  ```
  # primary option (mainline driver):
  insmod mac80211.ko
  insmod rtl8xxxu.ko
  ip -br link          # a new wlanX should appear
  ```
  If `rtl8xxxu` doesn't drive it, we try `insmod 8192eu.ko` instead.
  Nothing here is permanent — a reboot clears any loaded module.

---

## If the build fails (red ✗)
Open the failed run, tap the failed step, copy the red error text, and paste it back to me.
CI builds like this often need one or two small fixes — I'll patch the workflow and you
re-run Step 4. The most likely spots are the out-of-tree `8192eu.ko` step (it's allowed to
fail without stopping the rest) or a kernel API tweak.

## Honest expectations
- `rtl8xxxu.ko` is matched to your exact kernel (same source commit, same compiler, same
  vermagic) so it *should* load. There's still a small chance the kernel refuses an unsigned
  module or a CFI check trips — if so, a reboot fixes it and we adjust.
- This only gets the dongle working as a network interface. Monitor mode specifically depends
  on `8192eu.ko` building successfully.
