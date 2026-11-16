# BuckThem Domains — Capacitor iOS shell BUILD-RECORD

- Slug: `domains` · Shell: `~/workspace/testflight-all-products-2026-11-16/shells/domains/`
- Built: 2026-11-16 (local work only — no GitHub push, no server change, no iMac use)
- Recipe: estate All-Apps pattern (feedtosold build record + town-shell template)

## Identity

- Display name: **BuckThem Domains**
- Display-name source: lane record ~/workspace/domain-store-2026-11-15/STATUS.md ("Store name: BuckThem Domains") + live manifest name "BuckThem Domains"
- Bundle ID: `com.buckthem.domains`
- App surface: https://domains.buckthem.com/
- Live check: GET https://domains.buckthem.com/ via proxy http://127.0.0.1:18080 -> HTTP 200 (subdomain resolves and serves; DNS was added tonight by another lane)

## Shell scaffold

- `package.json`: copied from town-shell, only the `name` field changed (to `domains-shell`).
  Resolved versions (read back from package-lock.json): Capacitor core/cli/ios 6.2.3, @capacitor/share 6.0.4, @capacitor/push-notifications 6.0.5.
- `capacitor.config.json`: appId `com.buckthem.domains`, appName `BuckThem Domains`, backgroundColor `#ffffff`, ios contentInset `always` / preferredContentMode `mobile`, server androidScheme `https`, cleartext false, url `https://domains.buckthem.com/`, webDir `www`.
- `www/index.html`: redirect stub to https://domains.buckthem.com/ (town-shell pattern).
- Install route: node_modules + package-lock.json carried from town-shell's installed tree via hardlink copy (cp -al), file count verified 4629/4629. See Flags for why.
- iOS scaffold route: assembled from town-shell's proven ios tree via hardlink copy (cp -al) with product-specific files replaced and read back. See Flags for why.
- `IPHONEOS_DEPLOYMENT_TARGET` set to 15.0 (all four configs) — deliberate deviation from the town template's 13.0, which the parent program's STATUS already flags as below Xcode 27.1's 15.0 floor with the same fix staged for Town.

## Icon / splash

- Icon source: the product's live PWA manifest icon (https://domains.buckthem.com/icon.svg, the product's own SVG mark) fetched via the proxy and rasterized to 1024x1024 PNG with sharp (43,301 bytes).
- `resources/icon.png`: 1024x1024. `resources/splash.png`: 2732x2732 (icon centered 1200x1200 on white, PIL).
- iOS AppIcon: `AppIcon.appiconset` in the single-size format — `AppIcon-512@2x.png` (1024x1024, hardlink of resources/icon.png) + town-shell's `Contents.json`.
- Splash.imageset PNGs replaced with this product's splash (hardlinks).

## Read-backs (exact)

- pbxproj `PRODUCT_BUNDLE_IDENTIFIER`: ["com.buckthem.domains", "com.buckthem.domains"] (Debug + Release) — MATCH
- Info.plist `CFBundleDisplayName`: `BuckThem Domains` — MATCH
- capacitor.config.json parses: appId `com.buckthem.domains`, server.url `https://domains.buckthem.com/` — MATCH; the synced `ios/App/App/capacitor.config.json` carries the same appId/url plus packageClassList [PushNotificationsPlugin, SharePlugin]
- Usage strings: NONE found in ios/App/App/ (grep UsageDescription) — correct for a web shell

## Flags / gaps

- SANDBOX WRITE DEGRADATION (affects method, not result): during this task the VM's storage layer intermittently-to-consistently refused content writes from exec processes (EPERM at write; files landing as 0-byte stubs; rsync writes always refused; npm's own log writes refused). Consequences, all worked around and verified by read-back:
  - The recipe's /tmp npm staging succeeded once (the first domains attempt) and was refused thereafter; node_modules was therefore carried from town-shell's installed tree (identical dependency set — the shells' package.json differs from town-shell's only in the `name` field) via hardlink copy. Hardlinks share inodes with town-shell's tree — safe here because cap tooling only reads node_modules, node_modules is never committed, and a fresh `npm install` on the Mac replaces it. The package-lock.json hardlink's root `name` field reads `town-shell` (cosmetic; dependency versions verified above).
  - `cap add ios` / `cap sync ios` were attempted (add rc=1, sync rc=1 on the final pass) and produced a hollow tree of 0-byte stubs; the ios tree was therefore assembled from town-shell's proven tree via hardlink copy, with the product's pbxproj (bundle ID + deployment target 15.0), Info.plist (display name), synced capacitor.config.json, public/index.html, AppIcon, and splash images replaced and every value read back (above). On the Mac, a fresh `npm install` + `npx cap sync ios` in a normal environment will regenerate this tree from the committed sources.
- iOS compilation / signing / TestFlight upload are Mac steps (Tylers-iMac queue) — not done here, per task scope.
- App Store Connect app-record creation remains the account holder's portal step — not done here.
