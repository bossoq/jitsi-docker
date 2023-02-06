ARG JITSI_REPO=bossoq
FROM ${JITSI_REPO}/base-java

RUN \
	apt-dpkg-wrap apt-get update && \
	apt-dpkg-wrap apt-get install -y jitsi-videobridge2 jq curl iproute2 dnsutils && \
	apt-cleanup

COPY rootfs/ /

VOLUME /config

ENTRYPOINT [ "/init" ]