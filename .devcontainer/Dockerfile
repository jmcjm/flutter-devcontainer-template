# --- Builder Stage ---
FROM --platform=linux/amd64 debian:bookworm-slim AS builder

# Avoid warnings by switching to noninteractive
ARG DEBIAN_FRONTEND=noninteractive

ENV ANDROID_SDK_ROOT=/android-sdk
ENV FLUTTER_SDK_ROOT=/flutter

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    wget \
    unzip \
    xz-utils \
    openjdk-17-jdk \
    gpg \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Download and install Android Command Line Tools
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools \
    && wget -O commandlinetools.zip "https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip" \
    && unzip commandlinetools.zip -d ${ANDROID_SDK_ROOT}/cmdline-tools \
    && mv ${ANDROID_SDK_ROOT}/cmdline-tools/cmdline-tools ${ANDROID_SDK_ROOT}/cmdline-tools/latest \
    && rm commandlinetools.zip

# Accept Android SDK licenses and install necessary components
RUN yes | ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager --licenses && \
    ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager "build-tools;34.0.0" "cmake;3.22.1" "ndk;26.3.11579264" "platform-tools" "platforms;android-35"

# Download and install Flutter
RUN mkdir -p ${FLUTTER_SDK_ROOT} \
    && wget -O flutter.tar.xz "https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.22.2-stable.tar.xz" \
    && tar xf flutter.tar.xz -C ${FLUTTER_SDK_ROOT} --strip-components=1 \
    && rm flutter.tar.xz

# --- Final Stage ---
FROM --platform=linux/amd64 debian:bookworm-slim AS final

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG HOME=/home/${USERNAME}

ENV ANDROID_SDK_ROOT=${HOME}/android-sdk
ENV FLUTTER_SDK_ROOT=${HOME}/flutter
ENV PATH=$PATH:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin
ENV PATH=$PATH:${ANDROID_SDK_ROOT}/platform-tools
ENV PATH=$PATH:${FLUTTER_SDK_ROOT}/bin
ENV SHELL=/bin/zsh

# Install dependencies including Google Chrome
RUN dpkg --add-architecture amd64 \
    && apt-get update \
    && apt-get -y install --no-install-recommends \
    curl \
    wget \
    git \
    unzip \
    xz-utils \
    zip \
    openjdk-17-jdk \
    libglu1-mesa \
    libc6:amd64 \
    sudo \
    zsh \
    ca-certificates \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    libgtk-3-dev \
    liblzma-dev \
    nano \
    build-essential \
    gpg \
    # Add Google Chrome
    && wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends google-chrome-stable \
    # End Google Chrome installation
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /etc/apt/sources.list.d/google-chrome.list # Clean up the added repo file

# Create a non-root user
RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} -m -s /bin/zsh ${USERNAME} \
    && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

USER ${USERNAME}
WORKDIR ${HOME}

# Copy SDKs from the builder stage to the user's home directory
COPY --from=builder --chown=${USERNAME}:${USERNAME} /android-sdk ${ANDROID_SDK_ROOT}
COPY --from=builder --chown=${USERNAME}:${USERNAME} /flutter ${FLUTTER_SDK_ROOT}
