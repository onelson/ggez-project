FROM theomn/rustup-base:latest


RUN apt-get update \
    && apt-get install --no-install-recommends -y \
      sudo \
      pkg-config \
      libasound-dev \
      libsdl2-dev \
      mesa-utils \
      libgl1-mesa-dri \
      libgl1-mesa-glx \
      alsa-utils \
    && rm -rf /var/lib/apt/lists/*

# Once all the package/lib installs are complete, configure the local user
# and switch to userland.
ARG DEVELOPER_UID
ARG DEVELOPER_GID

# Make a user and group to match the user running the build
RUN groupadd -g ${DEVELOPER_GID} developer \
    && useradd -u ${DEVELOPER_UID} \
               -g ${DEVELOPER_GID} \
               -G users \
               -G audio \
               -G sudo \
               --create-home developer \
    && chown -R developer:developer /rust \
    && echo "developer ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/00-developer

# !!! Anything requiring root should happen before this point.
USER developer

VOLUME /code
WORKDIR /code

# FIXME: This cheap trick basically tells alsa not to try to bind to hardware.
# I have not yet managed to figure out how to share the audio from the host
# into the container...
RUN echo "pcm.!default = null;" > /home/developer/.asoundrc
