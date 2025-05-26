FROM ubuntu:25.04

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

# Configure locales
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen
ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8"

# Create application directory and set it as working directory
RUN mkdir -p /app
WORKDIR /app

# Define Reaper download URL as a build argument
ARG REAPER_URL="https://dlcf.reaper.fm/7.x/reaper739_linux_x86_64.tar.xz"

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
