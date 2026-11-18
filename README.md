# UNMAINTAINED. NO SUPPORT.

**Issues and pull requests are disabled.** This is a one-off snapshot of a prototype, not a project that will be maintained.

**Use at your own risk.** There is no warranty of any kind, express or implied, and no liability for damage to hardware, software or data, or for any other loss, arising from its use. The same MIT OR Apache-2.0 licence terms as FilmCraft apply (`LICENSE-MIT`, `LICENSE-APACHE`).

---

# FilmCraft: External Video Output on a DeckLink card (Linux)

A short series of commits for [storytold/filmcraft](https://github.com/storytold/filmcraft) that sends the Program monitor to a Blackmagic DeckLink card on Linux.

## What it does

- **Linux only.** On other platforms the setting stays greyed out.
- **DeckLink output:** device 0, 1920×1080, 25p, 8-bit Y'CbCr 4:2:2 (UYVY), BT.709 limited range.
- **Gamma switch** (Settings ▸ Playback ▸ External Video Output gamma): `srgbToBt1886` (default: sRGB EOTF to linear, re-encoded with gamma 1/2.4 for a BT.1886 display; matched Avid Media Composer on the same TV) or `asIs` (sRGB values unchanged). Switchable live.
- **Frame snapshot** of the UYVY buffer last pushed to the card (after the gamma LUT): raw `.uyvy`, a `.json` sidecar, and a full-range RGB `.png`. From Settings ▸ Playback, or the control-bridge command `transmit.snapshot` (optional path). Default folder: `~/Pictures/filmcraft-decklink-snapshots`.

A longer guide, including requirements, limitations and tested hardware, is in `decklink-output.md`.

## Build against upstream `70dd1a53`

Base: [storytold/filmcraft](https://github.com/storytold/filmcraft) at `70dd1a53` (`engine, ui: default screenshot, session log and voice-over folders go in a private temp folder (#4264)`).

Apply the patches:

```sh
git clone https://github.com/storytold/filmcraft && cd filmcraft
git checkout -b decklink-output 70dd1a53
git am /path/to/decklink-publish/patches/*.patch
```

On a newer `main`, try `git am -3 …`. The only file both sides are likely to touch is `Cargo.lock` (one added line: `cc` in `filmcraft-platform`'s dependencies). If it conflicts, take `main`'s `Cargo.lock`, run `cargo build` and `git add Cargo.lock`, then `git am --continue`.

`git am` warns about trailing whitespace in `0001`: that is in Blackmagic's SDK headers, which are kept byte-for-byte unmodified. Do not use `--whitespace=fix`.

Or fetch from the bundle (a clone that already has `70dd1a53`):

```sh
git bundle verify /path/to/decklink-output.bundle
git fetch /path/to/decklink-output.bundle decklink-output-anon:decklink-output
git checkout decklink-output
```

Then:

```sh
cargo build --release -p filmcraft -p filmcraft-cli
./target/release/filmcraft-cli decklink-test --hold 30
./target/release/filmcraft   # Settings ▸ Playback ▸ Enable External Video Output
```

A C++ compiler (`g++`) is required in addition to FilmCraft's usual build dependencies. Running the output needs Blackmagic Desktop Video and a DeckLink card; building does not.

## Contents

| File | |
|---|---|
| `patches/000N-*.patch` | `git format-patch` series from branch `decklink-output-anon` |
| `decklink-output.bundle` | git bundle of that branch (requires `70dd1a53`) |
| `decklink-output.md` | the guide (same as `docs/decklink-output.md` in the series) |
| `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE` | copied from FilmCraft |
| `SHA256SUMS` | checksums of the files in this repository |
| `COMMENT_2174.txt` | draft comment for [storytold/filmcraft#2174](https://github.com/storytold/filmcraft/issues/2174) |

The seven commits:

1. `platform: DeckLink video output (Linux, 1080p25 UYVY)`
2. `gpu: ExportRenderer::render_rgba8 / composite_rgba8`
3. `ui: External Video Output on a DeckLink card (Linux)`
4. `cli: decklink-test and decklink-bench`
5. `docs: External Video Output on a DeckLink card`
6. `decklink: External Video Output gamma (sRGB to Rec.709 BT.1886 / As is)`
7. `decklink: snapshot the frame last sent to the card`

## Licences

FilmCraft's own code is MIT OR Apache-2.0. The vendored DeckLink SDK 12.0 headers (`crates/platform/decklink/sdk/`) are Blackmagic Design's under the Boost Software License 1.0 (notice kept in each file; listed in `NOTICE`).
