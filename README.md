# reaper_docker

This repository provides a Dockerized version of REAPER (Rapid Environment for Audio Production, Engineering, and Recording), a powerful and flexible Digital Audio Workstation (DAW).

## What it Does

The Docker image encapsulates REAPER along with all its necessary dependencies, allowing you to run REAPER in an isolated and consistent environment. It is pre-configured to use PipeWire-JACK for audio routing.

## Advantages

- **Dependency Management:** All required libraries and dependencies for REAPER are bundled within the Docker image. This eliminates the need to install them manually on your host system and avoids potential conflicts with existing system libraries.
- **Portability:** Run REAPER consistently across any system that has Docker installed, regardless of the underlying operating system or its configuration.
- **Reproducibility:** Guarantees a stable and reproducible environment for your audio projects. Every time you run the container, you get the same setup.
- **Isolation:** REAPER runs in an isolated container, preventing interference with or from other applications on your host system.
- **PipeWire-JACK Integration:** The image is configured to launch REAPER with `pw-jack`, enabling seamless integration with the modern PipeWire audio server and JACK applications. Importantly, this setup utilizes the host machine's PipeWire instance, allowing audio to be shared and routed between the containerized REAPER and other applications running on the host.
- **Easy Updates:** The Dockerfile can be easily modified to use different versions of REAPER by changing the `REAPER_URL` build argument.

## How to Use

### Prerequisites

- Docker installed on your system.
- Git (for cloning the repository and for the build script to determine `GIT_COMMIT`).
- An X server running on your host (for the GUI).
- PipeWire installed and running on your host.

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
    This will create a Docker image tagged as `reaper:latest`. The script uses `docker/Dockerfile` and passes the current git commit hash as a build argument.

### Running REAPER

1.  Execute the run script:
    ```bash
    ./docker/run
    ```
    This script will:
    - Allow connections to the X server using `xhost +`.
    - Run the `reaper:latest` Docker image with necessary configurations for audio, display, and user permissions.
    - Mount PipeWire and PulseAudio sockets from the host to enable audio sharing.
    - Set the working directory inside the container to `/app`.
    - **Important for file access:** The script mounts the directory from which it is run (your current working directory on the host) to `/cwd` inside the container. This allows REAPER to access projects and files from that location. You can save your REAPER projects in this directory or its subdirectories to easily access them both inside and outside the container.

### Customizing the REAPER Version

To use a different version of REAPER, you can modify the `REAPER_URL` build argument in the `docker/Dockerfile` or pass it during the build command:
```bash
docker build \
    --progress=plain \
    --network=host \
    -t reaper:latest \
    --build-arg GIT_COMMIT=$(git rev-parse --short HEAD) \
    --build-arg REAPER_URL="<new_reaper_download_url>" \
    -f docker/Dockerfile .
```
Or, more simply, by modifying the `REAPER_URL` directly in the `docker/Dockerfile` and then running the `./docker/build` script.
