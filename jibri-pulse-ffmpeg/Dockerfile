ARG JITSI_REPO=bossoq
ARG BASE_TAG=latest
FROM ${JITSI_REPO}/base-java-ffmpeg:${BASE_TAG}

ARG TARGETPLATFORM=linux/amd64
ARG USE_CHROMIUM=0
#ARG CHROME_RELEASE=latest
#ARG CHROMEDRIVER_MAJOR_RELEASE=latest
ARG CHROME_RELEASE=109.0.5414.74
ARG CHROMEDRIVER_MAJOR_RELEASE=109

RUN mv /usr/local/bin/ffmpeg /usr/local/bin/ffmpeg-original

COPY rootfs/ /

RUN apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y fontconfig && \
    apt-dpkg-wrap apt-get install -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" jibri libgl1-mesa-dri procps jitsi-upload-integrations jq pulseaudio dbus dbus-x11 rtkit unzip && \
    /usr/bin/install-chrome.sh && \
    apt-cleanup && \
    adduser jibri rtkit

VOLUME /config