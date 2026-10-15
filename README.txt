ZeoPlant mobile app (Progressive Web App)
==========================================
Files
  index.html            the calculator, phone-first layout (also works opened directly on a PC)
  manifest.webmanifest  app name, icon, standalone (no browser bars) when installed
  sw.js                 offline cache: once opened, works without network
  icon.svg / icon-*.png app icons (home screen / splash)

To install on a phone the folder must be served over HTTPS (any static host:
Netlify Drop, GitHub Pages, Cloudflare Pages, your own site). Upload the whole
folder, open the link on the phone, then:
  Android / Chrome : menu (three dots) > "Add to Home screen" / "Install app"
  iPhone / Safari  : Share > "Add to Home Screen"

Opening index.html as a plain file (USB, WhatsApp, e-mail) also works on
Android Chrome, just without the install and offline features.

Language: EN / FR / DE switcher in the header (chip bar on phones); first visit follows the phone language,
the choice is remembered on the device (localStorage zeoplant.lang).
Inputs are remembered on the device (localStorage). "Reset to Excel defaults" clears them.
After editing index.html, change CACHE in sw.js (zeoplant-v1 -> v2) so phones pick up the update.
Source: ../ZeoPlant-Dashboard.html (unchanged). Rebuild script: see Claude session scratchpad build.js.
