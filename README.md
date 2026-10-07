# DXMT cross-process swapchains

[DXMT](https://github.com/3Shain/dxmt) **v0.80** plus one change: Direct3D 11 swapchains can present
to a window owned by another process. This fixes the **black Steam window** when running the
Windows Steam client through Wine and DXMT on macOS.

Requires the matching Wine branch:
**[jacksonking109/wine `dxmt-macdrv`](https://github.com/jacksonking109/wine/tree/dxmt-macdrv)**.

Hobby project, tested on Apple Silicon with x86_64 Wine 11.18 under Rosetta 2.

## What changed

Steam's UI is Chromium, whose GPU process renders into a window owned by the browser process.
DXMT rejected that (`CreateSwapChain: cross-process swapchain not supported yet`).

- **DXMT:** `CreateSwapChain` no longer refuses such windows; Wine decides.
- **Wine:** `winemac.drv` exports the `macdrv_functions` table DXMT looks for (so DXMT works on
  current Wine), and backs other-process windows with Wine's existing remote-layer swapchain
  (`CAContext` + `CALayerHost`).

## Build

1. Build Wine from `dxmt-macdrv` for x86_64 (`CC="clang -arch x86_64"`,
   `--enable-archs=i386,x86_64`). Steam needs GnuTLS, so build it with GnuTLS available.
2. Install the official [DXMT v0.80](https://github.com/3Shain/dxmt/releases/tag/v0.80) release into it.
3. Build this branch's `d3d11.dll` and replace the release's copy, in Wine's
   `lib/wine/x86_64-windows` **and** in each prefix's `system32`:

   ```sh
   meson setup --cross-file build-win64.txt -Dwine_build_path=<wine build dir> build64
   ninja -C build64 src/d3d11/d3d11.dll
   ```

   `d3d11.dll` doesn't need LLVM, but it does need Xcode's `metal` compiler. Without Xcode, reuse
   the `dxmt_command.metallib` embedded in the official v0.80 `d3d11.dll`; the source is identical.
4. Start Steam with `-no-cef-sandbox` and without `-cef-disable-gpu`.

## Known limitations

- The layer keeps its initial size when the window is resized.
- Child windows not at the top-left of their top-level window are misplaced.

## License

MIT, like DXMT v0.80 (DXMT moved to LGPL after that release). The Wine branch is LGPL-2.1-or-later.

---

*Upstream DXMT README follows.*

# DXMT

A Metal-based translation layer for Direct3D 11 and 10 which allows running 3D applications on macOS using Wine.

For the current status of the project, please refer to the [project wiki](https://github.com/3Shain/dxmt/wiki).

The most recent development builds can be found [here](https://github.com/3Shain/dxmt/actions).


## Build

See [DEVELOPMENT.md](docs/DEVELOPMENT.md)
