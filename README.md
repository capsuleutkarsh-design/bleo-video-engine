# Bleo video engine

This is the playback engine of **Bleo**, a video player for Android and Windows (Gleo's sister app). It is
[mpv](https://mpv.io) and [FFmpeg](https://ffmpeg.org) built as one file, `libmpv.so`, for Android
(arm64-v8a, armeabi-v7a, x86_64; Android 8 / API 26 and up). Bleo puts it in place of the `libmpv.so` that comes
with `media_kit_libs_android_video`.

It's made the same way as Gleo's audio engine ([gleo-audio-engine](https://github.com/capsuleutkarsh-design/gleo-audio-engine)):
the same pinned sources, NDK and checks, with video switched on.

## What's in it

| | media_kit's default build | **Bleo's engine** |
|---|---|---|
| mpv / FFmpeg | 0.35 / 6.0 (2023) | **0.41 / 8.1.3** |
| Video decoders | a short list | **every FFmpeg decoder**: H.264, HEVC, AV1 (dav1d), VP8/9, ProRes, DNxHD, MPEG-2/4, VC-1, WMV… |
| Hardware decoding | MediaCodec | MediaCodec (H.264, HEVC, VP9, AV1 where the phone has it) |
| Audio | the common codecs | every FFmpeg decoder, incl. DTS, TrueHD, E-AC-3; soxr resampling |
| Subtitles | SRT, ASS | every FFmpeg subtitle decoder (ASS, SRT, WebVTT, PGS, DVD…) with libass |
| Filters | none | every LGPL FFmpeg filter (deinterlace with bwdif/yadif, rotate, scale…) |
| Screenshots | no image encoder | PNG and JPEG |
| Output | OpenGL ES (`vo=gpu`) | OpenGL ES (`vo=gpu`) and libplacebo (`vo=gpu-next`) |

- **Local files only:** FFmpeg opens `file` and Android `content://` URIs. Every network protocol is off. No scripting
  (Lua, JavaScript, C plugins).
- **Licence:** LGPL (`--enable-version3`, no `--enable-gpl`, nothing non-free). Each release has the corresponding
  source: every exact source tarball, the build scripts, the configure lines and all licences.
- **Checks in the build:** only Android system libraries needed; no text relocations; 16 KB page aligned (Android 15+);
  exports `mpv_lavc_set_java_vm` (media_kit calls it); and the codecs, filters and encoders above are really there.

## Build

GitHub Actions builds it on every push to `main` (the files are kept as a build artifact for 30 days), and makes a
release when run by hand: **Actions ▸ Build and release ▸ Run workflow**. Locally on Ubuntu:

```bash
sudo apt-get install -y nasm ninja-build pkg-config zip unzip python3-jinja2 cmake && pipx install meson==1.9.2
./build-android.sh      # -> out-android/
ABIS=arm64-v8a ./build-android.sh   # one ABI only
```

## Using it in Bleo

`VideoPlayer/android/app/build.gradle.kts` downloads a release zip, checks its SHA-256 and adds `lib/` as `jniLibs`.
To update: build a new release, then set its tag and the zip's SHA-256 (from `SHA256SUMS.txt`) there.

Made by Utkarsh Tripathi. mpv, FFmpeg and the libraries keep their own licences (see `LICENSES/` in each release).
