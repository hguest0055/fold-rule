# Fold Rule — Pixel 10 Pro Fold

An on-screen ruler sized from the panel's real pixel pitch, not the 96-CSS-px-per-inch fiction.

## Numbers it ships with

| | Cover panel | Inner panel |
|---|---|---|
| Resolution | 1080 × 2364 | 2076 × 2152 |
| Pixel density | 408 ppi | 373 ppi |
| Panel size | 67.2 × 147.2 mm | 141.4 × 146.5 mm |
| Body size | 76.3 × 155.2 mm (folded) | 150.4 × 155.2 mm (unfolded) |
| Bezel, sides | 4.5 mm | 4.5 mm |
| Bezel, top/bottom | 4.0 mm | 4.3 mm |
| Flat-area inset (default) | 1.5 mm | 2.5 mm |

Case lip defaults to 1.5 mm. Bezels are derived (body minus panel, halved); the flat inset and case
lip are estimates — trim them once against a caliper and they persist.

## Scale math

`css px per mm = ppi / 25.4 / devicePixelRatio × trim`

`devicePixelRatio` already folds in browser/page zoom, so zoom can't desync the scale. Panel choice
is auto-detected by comparing `screen.width × dpr` / `screen.height × dpr` against both panels'
native resolutions, so folding and unfolding re-scales on the fly.

## Zero points

- **On glass** — zero sits inset from the panel edge by the flat-area inset, past the curved glass,
  so an object laid flat on the screen starts on true flat. The dead strip is shaded.
- **Physical edge** — zero sits *off* the panel, out past the bezel and the case lip, at the table
  surface. The scale is cut off: the first visible tick is `bezel + case lip` mm, not 0.

## Rotation

Manifest asks for `portrait-primary` and the app calls `screen.orientation.lock()`. If the platform
refuses (browser tab without fullscreen), the whole stage is counter-rotated in CSS so the rulers
stay on the physical top and right edges regardless.

## Install

PWA install needs a secure origin. Either:

- push to a GitHub Pages repo and open the Pages URL in Chrome → menu → Install app; or
- serve locally and open it via `http://localhost` port-forwarded over ADB:
  `python3 -m http.server 8080` then `adb reverse tcp:8080 tcp:8080`.

Opening it as a `file://` URL works for measuring but won't install and won't register the service worker.

## AR measure

`ar.html`, reachable from the ruler screen or the app shortcut.

**Primary path — WebXR.** `immersive-ar` with `hit-test` required, plus `dom-overlay`,
`depth-sensing`, `anchors` and `light-estimation` as optional (the session is re-requested without
depth if the first attempt is rejected). Scale comes from ARCore's tracked geometry, the same source
Google's own Measure app uses. Points snap to hit-test poses; where no plane is found, the CPU depth
buffer is sampled at the crosshair and the point is placed along the reconstructed centre ray.
Rendering is raw WebGL — reticle, camera-facing line bands, vertices — with distances shown as
projected DOM labels. Path mode chains segments and totals them; Pairs mode measures independently.

**Requirements:** Chrome on Android, HTTPS or localhost, and Google Play Services for AR installed.
None of it works from `file://`.

**Accuracy, honestly:** roughly 1–3% over a metre, degrading on dark, glossy or featureless
surfaces, and unreliable below about 5 cm. It drifts as you move. It is not a substitute for the
on-glass ruler on small parts — it exists for things too big to lay on the phone.

**Fallback — reference-object mode.** If WebXR is unavailable, a plain `getUserMedia` view lets you
tap the two ends of a known object (card, quarter, the phone's own 76.3 mm width) to derive mm/px,
then measure other things. This is only valid for objects in the same plane as the reference with
the camera square to that plane. Off-axis or at a different distance, it lies.
