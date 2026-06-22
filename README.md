# reaper_docker

A Dockerized build of [REAPER](https://www.reaper.fm/) (Rapid Environment for Audio
Production, Engineering, and Recording), a powerful and flexible Digital Audio
Workstation (DAW), bundled with a large collection of open-source audio plugins
and acoustic drum kits.

## What it Does

The Docker image packages REAPER together with all of its runtime dependencies, a
70+ package plugin library, and DrumGizmo drum kits, so you can run a fully loaded
DAW in an isolated, reproducible container. It is built on Ubuntu and configured to
use **PipeWire-JACK** for audio, sharing the **host machine's PipeWire instance** so
audio can be routed between the containerized REAPER and other applications on your
host.

## How it Works

- **Image (`docker/Dockerfile`):** Installs PipeWire/JACK, GStreamer, GTK and other
  dependencies; installs the plugin packages (see below); downloads DrumGizmo drum
  kits into `/usr/share/drumgizmo/kits`; and downloads and unpacks REAPER into
  `/app`. The container's default command launches REAPER through `pw-jack` so it
  connects to the host's PipeWire-JACK server.
- **Audio:** The host's PipeWire and PulseAudio sockets, plus `/dev/snd`, are mounted
  into the container. The container is granted `sys_nice` and a high `rtprio` limit
  so REAPER can use real-time scheduling for low-latency audio.
- **Display:** The host X server socket is mounted and `DISPLAY` is forwarded so the
  REAPER GUI appears on your desktop.
- **Identity & config:** The container runs as your host user/group (via `-u` and
  bind-mounted `/etc/passwd` and `/etc/group`), and your `~/.config/REAPER`
  directory is mounted in, so preferences, themes, and license persist across runs.
- **Networking:** The running container uses `--network=none` — REAPER has no network
  access at runtime, for isolation.

## Advantages

- **Dependency Management:** All required libraries and plugins are bundled in the
  image — nothing to install on the host, and no conflicts with host libraries.
- **Portability & Reproducibility:** The same setup runs on any Docker host and is
  identical every time you start the container.
- **Isolation:** REAPER runs in a container with no runtime network access.
- **Host PipeWire Integration:** Launched with `pw-jack` against the host's PipeWire,
  so audio can be shared and routed with other host applications.
- **Easy Version Updates:** Change the `REAPER_URL` build argument to build a
  different REAPER release (see below).

## Included Audio Plugins

The image comes with 70+ open-source audio plugin packages pre-installed across
multiple formats.

REAPER natively supports **LV2**, **VST**, **VST3**, and **CLAP** plugins on Linux.
LV2 plugins are auto-discovered via the `LV2_PATH` environment variable. VST/VST3
plugin paths can be configured in REAPER under Options → Preferences → Plug-ins → VST.

**LADSPA** and **DSSI** plugins are not natively supported by REAPER, but are
accessible through **Carla** (included as `carla-vst`), which acts as a plugin host
wrapper.

### LV2 (native)
Calf, LSP Plugins, x42, Dragonfly Reverb, Guitarix, ZynAddSubFX, ZaM Plugins, SWH, MDA, EQ10Q, Synthv1, Padthv1, Samplv1, Drumkv1, DrumGizmo, AVL Drums, Geonkick, SO Synth, setBfree, IR, LV2 Vocoder, Rubberband, BShapr, BSlizr, abGate, BSequencer, FOMP, DPF Plugins, BLOP, Bankstown

### VST / VST3 (native)
LSP Plugins, Dragonfly Reverb, IEM Plugin Suite, DPF Plugins, amsynth, ZynAddSubFX, Carla

### LADSPA (via Carla)
LSP Plugins, SWH, TAP, CAPS, CMT, DPF Plugins, BLOP, Autotalent, AMB, BlepVCO, FIL, REV, VCO, STE, WAH, bs2b, Omins, Rubberband

### DSSI (via Carla)
Hexter, WhySynth, WSynth, FluidSynth, DPF Plugins

## Included Drum Kits

High-quality multi-mic acoustic drum kits for **DrumGizmo**, downloaded into
`/usr/share/drumgizmo/kits`:

- Aasimonster
- DRSKit
- CrocellKit
- MuldjordKit

## How to Use

### Prerequisites

- Docker installed on your system.
- Git (for cloning the repository and for the build script to determine `GIT_COMMIT`).
- An X server running on your host (for the GUI).
- PipeWire (with PipeWire-JACK) installed and running on your host.

### Building the Image

1.  Clone this repository:
    ```bash
    git clone <repository_url>
    cd reaper_docker
    ```
2.  Run the build script:
    ```bash
    ./docker/build
    ```
    This builds an image tagged `reaper:latest` from `docker/Dockerfile`, passing the
    current git commit hash as the `GIT_COMMIT` build argument.

### Running REAPER

```bash
./docker/run
```

This script will:

- Run `xhost +` to allow the container to connect to your X server.
- Start the `reaper:latest` image with the audio, display, and user settings
  described in [How it Works](#how-it-works).
- Mount your **current working directory** to `/cwd` inside the container, so REAPER
  can read and save projects and files there (accessible both inside and outside the
  container).
- Mount your host `~/.config/REAPER` so settings persist.

#### Optional project/media mounts

You can expose additional host directories by setting environment variables before
running:

```bash
REAPER_PROJECTS_DIR=/path/to/projects \
REAPER_MEDIA_DIR=/path/to/media \
./docker/run
```

- `REAPER_PROJECTS_DIR` is mounted at `/proj`.
- `REAPER_MEDIA_DIR` is mounted at `/media`.

#### Optional pre/post hooks

You can run arbitrary host commands before the container starts and after it
exits by setting environment variables:

```bash
REAPER_PRE_HOOK='echo starting; my-setup-script' \
REAPER_POST_HOOK='my-cleanup-script' \
./docker/run
```

- `REAPER_PRE_HOOK` runs on the host before `docker run`.
- `REAPER_POST_HOOK` runs on the host after the container exits — including when
  REAPER is closed or the run is interrupted (e.g. Ctrl-C).

Both are optional and executed via `sh -c`.

### Customizing the REAPER Version

The REAPER version is controlled by the `REAPER_URL` build argument in
`docker/Dockerfile`. To build a different version, either edit `REAPER_URL` in the
Dockerfile and re-run `./docker/build`, or pass it directly:

```bash
docker build \
    --progress=plain \
    --network=host \
    -t reaper:latest \
    --build-arg GIT_COMMIT=$(git rev-parse --short HEAD) \
    --build-arg REAPER_URL="<new_reaper_download_url>" \
    -f docker/Dockerfile .
```
