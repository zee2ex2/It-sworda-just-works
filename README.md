# It sworda just works

Personal fork of [seedds/nxnative](https://github.com/seedds/nxnative), focused on **The Legend of Zelda: Ocarina of Time** (`0400486024DEC000`, v131072) at 60 fps. Branch: `oot6`.

## Fixes

- **Pool Finalize drops serial entries naming its textures** (ports upstream PR #2775). Live and ghost serials that resolve to a finalized pool's textures — directly or through a view's parent — are removed with the pool. Releases OoT's 12,288² shadow target, which stayed held by ~361 live + ~233 ghost serials (~594 MB).
- **Retire idle cache-only shared textures under macOS memory pressure** (ports upstream PR #2745). When resident memory exceeds 75% of `hw.memsize`, the oldest shared textures with no live/ghost serial, no pin, no GPU write and not render-target/shader-write are retired until below 65%. `mtl_mem_stat` logs `memory pressure: retired …`.
- **PlayStation / Xbox controller overlays** (ports upstream PR #2774). `DM_PAD_LAYOUT=playstation|xbox` swaps the pause-menu controller art, callout guides, external button glyphs, key legends and ocarina buttons.
- **Position-based button legends.** The Mac host remaps non-Nintendo pads by position (right/bottom/top/left), so legends now follow position: PlayStation ○/✕/△/□, Xbox B/A/Y/X. Xbox ocarina buttons are coloured (B red, A green, Y yellow, X blue).
- **Resolution overlay fixed.** The bundled `romfs_overlay-<scale>` pack was inert because nothing appended the mode; the launcher now selects `.../<docked|handheld>` from `DM_OPMODE` and prepends the controller-art overlay as a colon-separated `DM_ROMFS_OVERLAY` list.
- **Base already includes** the terrain-tile fix (#2077: Goron City translucent floor, Death Mountain holes) and the resolution-overlay variants.

## References

- Upstream: `seedds/nxnative` — base `decba4a0` on `oot6`, built on `6c3aeb37` (#2077 terrain fix).
- Ported pull requests (open/unmerged at fork time): **#2745**, **#2775**, **#2774**.
- Host position mapping: `dm/host/pad_apple.h` (`GCController` A/B/X/Y swapped by position for non-Nintendo pads).
- Game assets patched: `Font2/Scalable/External.Nin_Ounce_NVN2.bfotf.zs`; `UI/Layout2Archive/Common.Product.100.Nin_Ounce_NVN2.bntx.zs`.
- Controller art: `dm/games/0400486024DEC000/controller_dualsense.png`, `controller_xbox.png` (from PR #2774).
- Overlay generators: `scripts/oot_res_overlay.py`, `scripts/oot_pad_overlay.py` + `dm/tools/oot_pad_overlay.m`.
- Packaging: `scripts/build_ipa_from_nsp.py`, `dm/mac/pack_app.py`, `dm/mac/launcher.c`.
- Runtime env used: `DM_UPSCALER=metalfx`, `DM_ALIAS_SIZE=0x200000000`, `DM_FPS_HOLD=off`, `DM_NVN_UB_COPY_THREAD=0`, `DM_FOCUS_KEEP=1`.

## Dependencies

- **Game data (not included):** Ocarina of Time Switch NSP/NUP, title `0400486024DEC000`, version `131072`.
- **Python 3.14** virtualenv with `fontTools` (4.68), `zstandard` (or stdlib `compression.zstd` / `zstd` CLI), plus `lz4` and `capstone` for the other tooling.
- **.NET 10** (`dm/tools/nx_extract`).
- **Xcode command-line tools**: `clang`, Swift toolchain, macOS SDK.
- **Frameworks:** Foundation, CoreText, CoreGraphics, ImageIO, Metal, MetalFX, AppKit/SwiftUI.
- **macOS with Metal ASTC support** (the overlay tool re-encodes ASTC bntx textures on the GPU).

## Contributors

- **DeepSeek** — AI pair-programmer: diagnosis, code changes, build and verification.

## Build

```sh
cd "/Users/aaron/It sworda just works"
.venv/bin/python3 scripts/build_ipa_from_nsp.py \
  "<game>.nup" --platform mac-app \
  --work-dir "<build-dir>" --env-file "<build-dir>/mac_env.txt" \
  --allow-damaged -o "Ocarina of Time.app"
```
