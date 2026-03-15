FROM ubuntu:25.10

# Set frontend to noninteractive to avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        pipewire \
        pulseaudio-utils \
        pipewire-jack \
        libgtk-3-0 \
        libgstreamer1.0-0 \
        gstreamer1.0-plugins-base \
        gstreamer1.0-plugins-good \
        gstreamer1.0-plugins-bad \
        gstreamer1.0-plugins-ugly \
        gstreamer1.0-libav \
        gstreamer1.0-tools \
        gstreamer1.0-x \
        gstreamer1.0-alsa \
        gstreamer1.0-gl \
        gstreamer1.0-gtk3 \
        curl \
        libsm6 \
        locales \
        xz-utils \
        ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# Install audio plugins (LV2, LADSPA, VST, VST3, DSSI)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        # LV2 plugins
        calf-plugins \
        lsp-plugins-lv2 \
        x42-plugins \
        dragonfly-reverb-lv2 \
        guitarix-lv2 \
        zynaddsubfx-lv2 \
        zam-plugins \
        swh-lv2 \
        mda-lv2 \
        invada-studio-plugins-lv2 \
        eq10q \
        synthv1-lv2 \
        padthv1-lv2 \
        samplv1-lv2 \
        drumkv1-lv2 \
        drumgizmo \
        avldrums.lv2 \
        geonkick \
        so-synth-lv2 \
        setbfree \
        ir.lv2 \
        lv2vocoder \
        rubberband-lv2 \
        bshapr \
        bslizr \
        abgate \
        bsequencer \
        fomp \
        dpf-plugins-lv2 \
        blop-lv2 \
        bankstown-lv2 \
        # LADSPA plugins
        lsp-plugins-ladspa \
        swh-plugins \
        tap-plugins \
        caps \
        cmt \
        dpf-plugins-ladspa \
        invada-studio-plugins-ladspa \
        blop \
        autotalent \
        amb-plugins \
        blepvco \
        fil-plugins \
        rev-plugins \
        vco-plugins \
        ste-plugins \
        wah-plugins \
        bs2b-ladspa \
        omins \
        rubberband-ladspa \
        # VST/VST3 plugins
        lsp-plugins-vst \
        lsp-plugins-vst3 \
        dragonfly-reverb-vst \
        dragonfly-reverb-vst3 \
        iem-plugin-suite-vst \
        iem-plugin-suite-vst3 \
        dpf-plugins-vst \
        dpf-plugins-vst3 \
        amsynth \
        zynaddsubfx-vst \
        carla-vst \
        # DSSI plugins
        hexter \
        xsynth-dssi \
        whysynth \
        wsynth-dssi \
        fluidsynth-dssi \
        dpf-plugins-dssi \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# Install unzip and create drumgizmo kits directory
RUN apt-get update && \
    apt-get install -y --no-install-recommends unzip && \
    mkdir -p /usr/share/drumgizmo/kits && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# Download DrumGizmo drum kits (high-quality multi-mic acoustic drums)
# Each kit is a separate layer so failures don't require re-downloading everything.
# Using a resume loop (curl -C -) so retries continue where they left off instead
# of restarting, which avoids re-downloading already-transferred data.
RUN until curl -L -C - -o /tmp/aasimonster.zip \
    "https://drumgizmo.org/kits/Aasimonster/aasimonster2_1.zip"; do sleep 10; done && \
    unzip -q /tmp/aasimonster.zip -d /usr/share/drumgizmo/kits/ && \
    rm /tmp/aasimonster.zip

RUN until curl -L -C - -o /tmp/drskit.zip \
    "https://drumgizmo.org/kits/DRSKit/DRSKit2_1.zip"; do sleep 10; done && \
    unzip -q /tmp/drskit.zip -d /usr/share/drumgizmo/kits/ && \
    rm /tmp/drskit.zip

RUN until curl -L -C - -o /tmp/crocellkit.zip \
    "https://drumgizmo.org/kits/CrocellKit/CrocellKit1_1.zip"; do sleep 10; done && \
    unzip -q /tmp/crocellkit.zip -d /usr/share/drumgizmo/kits/ && \
    rm /tmp/crocellkit.zip

RUN until curl -L -C - -o /tmp/muldjordkit.zip \
    "https://drumgizmo.org/kits/MuldjordKit/MuldjordKit3.zip"; do sleep 10; done && \
    unzip -q /tmp/muldjordkit.zip -d /usr/share/drumgizmo/kits/ && \
    rm /tmp/muldjordkit.zip

# LV2 discovery via Lilv library (used by Reaper for LV2 plugin scanning)
ENV LV2_PATH="/usr/lib/lv2:/usr/lib/x86_64-linux-gnu/lv2"

# Configure locales
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen
ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8"

# Create application directory and set it as working directory
RUN mkdir -p /app
WORKDIR /app

# Define Reaper download URL as a build argument
ARG REAPER_URL="https://www.reaper.fm/files/7.x/reaper755_linux_x86_64.tar.xz"

# Download and install Reaper
# Using -L for curl to follow redirects
RUN curl -L -o reaper.tar.xz ${REAPER_URL} && \
    tar -xJf reaper.tar.xz && \
    rm reaper.tar.xz
    # The tarball is expected to extract a directory named 'reaper_linux_x86_64'
    # containing the REAPER application.

# Arguments and Labels
ARG HOME
ARG GIT_COMMIT=unspecified
LABEL revision=$GIT_COMMIT
LABEL reaper.url="${REAPER_URL}"

# Run Reaper with PipeWire-JACK
CMD ["pw-jack", "/app/reaper_linux_x86_64/REAPER/reaper"]
