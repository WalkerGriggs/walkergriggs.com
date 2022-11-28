+++
title = "Pipewire in Docker"
author = ["Walker Griggs"]
date = 2022-12-03
categories = ["devlog"]
draft = false
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2011
+++

{{< figure src="/pipewire-in-docker/pipewire.gif" width="100%" >}}

[Pipewire](https://pipewire.org/) is a graph-based multimedia processing engine that lets you handle audio + video in real time! I've had way too much fun playing with it recently, but spent longer than I care to admit spinning it up in an Ubuntu container.

Most of the examples I saw floating around were using [systemd](https://www.freedesktop.org/wiki/Software/systemd/) or [Fedora](https://getfedora.org/en/server/), but my requirements were

1.  Ubuntu 22.04
2.  Processes run as background sub-shells without systemd
3.  Built from the latest source
4.  Drop-in replacement for PulseAudio

Side note: I spent some time tinkering with 18.04 LTS, which requires either a [PPA](https://pipewire-debian.github.io/pipewire-debian/) or building [Meson](https://mesonbuild.com/Reproducible-builds.html) and [Alsa utils](https://github.com/alsa-project/alsa-utils) from scratch (Pipewire requires versions not available older Debian systems). I highly recommend the PPA if you head that route...


## Front matter and dependencies {#front-matter-and-dependencies}

As with most containers, we first define the front matter and install all Pipewire build / runtime dependencies. There are probably a few unnecessary packages floating around here, but the goal of this spike wasn't to optimize the container's size.

```Dockerfile
FROM ubuntu:22.04 AS pw_build

LABEL description="Ubuntu-based stage for building pipewire" \
      maintainer="Walker Griggs <walker@walkergriggs.com>"

RUN apt-get update \
    && apt-get install -y \
    debhelper-compat \
    findutils        \
    git              \
    libasound2-dev   \
    libdbus-1-dev    \
    libglib2.0-dev   \
    libsbc-dev       \
    libsdl2-dev      \
    libudev-dev      \
    libva-dev        \
    libv4l-dev       \
    libx11-dev       \
    ninja-build      \
    pkg-config       \
    python3-docutils \
    python3-pip      \
    meson            \
    pulseaudio       \
    dbus-x11         \
    rtkit            \
    xvfb
```


## Relevant environment variables {#relevant-environment-variables}

The next step is setting the relevant environment variables for building Pipewire. I like to do this after installing dependencies so I don't have to re-install everything if one variable changes.

In this example, we're pulling Pipewire's latest version (as of time of writing) and defining our build directory. We're building Pipewire in `/root` as `root` -- worst practice, but it's a spike.

```Dockerfile
ARG PW_VERSION=0.3.60
ENV PW_ARCHIVE_URL="https://gitlab.freedesktop.org/pipewire/pipewire/-/archive"
ENV PW_TAR_FILE="pipewire-${PW_VERSION}.tar"
ENV PW_TAR_URL="${PW_ARCHIVE_URL}/${PW_VERSION}/${PW_TAR_FILE}"

ENV BUILD_DIR_BASE="/root"
ENV BUILD_DIR="${BUILD_DIR_BASE}/build-$PW_VERSION"
```


## Build the thing {#build-the-thing}

Now that we've installed our dependencies, we're ready to build Pipewire itself. Meson is Pipewire's build system of choice. I don't have much experience with Meson, but it was easy enough to work with.

```Dockerfile
RUN curl -LJO $PW_TAR_URL \
    && tar -C $BUILD_DIR_BASE -xvf $PW_TAR_FILE

RUN cd $BUILD_DIR_BASE/pipewire-${PW_VERSION} \
    && meson setup $BUILD_DIR \
    && meson configure $BUILD_DIR -Dprefix=/usr \
    && meson compile -C $BUILD_DIR \
    && meson install -C $BUILD_DIR
```


## Setup the entrypoint scripts {#setup-the-entrypoint-scripts}

Next up are the dominoes of entrypoint scripts.

```Dockerfile
COPY startup/      /root/startup/
COPY entrypoint.sh /root/entrypoint.sh

WORKDIR /root
CMD ["/bin/bash", "entrypoint.sh"]
```

I like to breakdown the entrypoint scripts and order them with a filename prefix. I forget exactly where I picked up this habit, but it stuck a long time ago.

In this example, I'm running `xvfb` as a lightweight X11 server. From everything I've read, Pipewire is really designed to run on a full `Wayland` system, but I haven't made the jump on any of my machines and likely wont for some time.

```bash
# startup/00_try-sh.sh
for f in startup/*; do
    source "$f" || exit 1
    sleep 2s
done

# startup 01_envs.sh
export DISABLE_RTKIT=y
export XDG_RUNTIME_DIR=/tmp
export PIPEWIRE_RUNTIME_DIR=/tmp
export PULSE_RUNTIME_DIR=/tmp
export DISPLAY=:0.0

# startup/10_dbus.sh
mkdir -p /run/dbus
dbus-daemon --system --fork

# startup/20_xvfb.sh
Xvfb -screen $DISPLAY 1920x1080x24 &

# startup/30_pipewire.sh
mkdir -p /dev/snd
pipewire &
pipewire-media-session &
pipewire-pulse &
```

Pipewire has a few runtime requirements; [dbus](https://www.freedesktop.org/wiki/Software/dbus/) and [rtkit](https://github.com/heftig/rtkit) are top of mind. So long as the Pipewire media session can fork the system dbus session though (or launch a new one), you should be fine. I've personally disabled rtkit.

Another point of note: I've opted for [media-session](https://gitlab.freedesktop.org/pipewire/media-session) which is, unsurprisingly, a reference implementation of Pipewire's media session. In future revisions, I plan to replace it with the more advanced [Wireplumber](https://gitlab.freedesktop.org/pipewire/wireplumber). Media Session was quick and easy for the time being though.


## Run the thing! {#run-the-thing}

There's not much to it. If we hop into the container and check on the Pulse server's, we can see that our Pipewire server is running and properly emulating Pulse. Great success!

```text
root@8e86f658e342:/# pactl info
Server String: /tmp/pulse/native
Library Protocol Version: 35
Server Protocol Version: 35
Is Local: yes
Client Index: 42
Tile Size: 65472
User Name: root
Host Name: 8e86f658e342
Server Name: PulseAudio (on PipeWire 0.3.59)
Server Version: 15.0.0
Default Sample Specification: float32le 2ch 48000Hz
Default Channel Map: front-left,front-right
```

I'll likely write more about Pipewire once I get more experiencing working with it as a desktop service and as an API client. [Wim](https://hachyderm.io/@wtay@fosstodon.org) and [team](https://hachyderm.io/@pipewire@fosstodon.org) have written some great [client examples](https://docs.pipewire.org/examples.html) which I've modified for a few different use cases -- the [Simple Plugin API (SPA)](https://docs.pipewire.org/page_spa.html) is surprisingly... simple. More to follow!
