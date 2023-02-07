ARG JITSI_REPO=bossoq
FROM ${JITSI_REPO}/base-java AS base

WORKDIR /tmp/workdir

RUN     apt-get -yqq update && \
    apt-get install -yq --no-install-recommends ca-certificates expat libgomp1 && \
    apt-get autoremove -y && \
    apt-get clean -y

FROM base as build

ENV     FFMPEG_VERSION=5.1.2 \
    FDKAAC_VERSION=0.1.5 \
    LAME_VERSION=3.100 \
    LIBPTHREAD_STUBS_VERSION=0.4 \
    LIBXCB_VERSION=1.13.1 \
    XCBPROTO_VERSION=1.13 \
    OGG_VERSION=1.3.2 \
    VORBIS_VERSION=1.3.5 \
    VPX_VERSION=1.8.0 \
    X264_VERSION=20170226-2245-stable \
    X265_VERSION=3.4 \
    XAU_VERSION=1.0.9 \
    XORG_MACROS_VERSION=1.19.2 \
    XPROTO_VERSION=7.0.31 \
    LIBPNG_VERSION=1.6.9 \
    SRC=/usr/local

ARG         OGG_SHA256SUM="e19ee34711d7af328cb26287f4137e70630e7261b17cbe3cd41011d73a654692 libogg-1.3.2.tar.gz"
ARG         VORBIS_SHA256SUM="6efbcecdd3e5dfbf090341b485da9d176eb250d893e3eb378c428a2db38301ce libvorbis-1.3.5.tar.gz"


ARG         LD_LIBRARY_PATH=/opt/ffmpeg/lib
ARG         MAKEFLAGS="-j2"
ARG         PKG_CONFIG_PATH="/opt/ffmpeg/share/pkgconfig:/opt/ffmpeg/lib/pkgconfig:/opt/ffmpeg/lib64/pkgconfig"
ARG         PREFIX=/opt/ffmpeg
ARG         LD_LIBRARY_PATH="/opt/ffmpeg/lib:/opt/ffmpeg/lib64"


ARG DEBIAN_FRONTEND=noninteractive

RUN      buildDeps="autoconf \
    automake \
    cmake \
    curl \
    bzip2 \
    libexpat1-dev \
    g++ \
    gcc \
    git \
    gperf \
    libtool \
    make \
    meson \
    nasm \
    perl \
    pkg-config \
    python \
    libssl-dev \
    yasm \
    libva-dev \
    zlib1g-dev" && \
    apt-get -yqq update && \
    apt-get install -yq --no-install-recommends ${buildDeps}
## x264 http://www.videolan.org/developers/x264.html
RUN \
    DIR=/tmp/x264 && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://download.videolan.org/pub/videolan/x264/snapshots/x264-snapshot-${X264_VERSION}.tar.bz2 | \
    tar -jx --strip-components=1 && \
    ./configure --prefix="${PREFIX}" --enable-shared --enable-pic --disable-cli && \
    make && \
    make install && \
    rm -rf ${DIR}
### x265 http://x265.org/
RUN \
    DIR=/tmp/x265 && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://github.com/videolan/x265/archive/refs/tags/${X265_VERSION}.tar.gz | \
    tar -zx && \
    cd x265-${X265_VERSION}/build/linux && \
    sed -i "/-DEXTRA_LIB/ s/$/ -DCMAKE_INSTALL_PREFIX=\${PREFIX}/" multilib.sh && \
    sed -i "/^cmake/ s/$/ -DENABLE_CLI=OFF/" multilib.sh && \
    ./multilib.sh && \
    make -C 8bit install && \
    rm -rf ${DIR}
### libogg https://www.xiph.org/ogg/
RUN \
    DIR=/tmp/ogg && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO http://downloads.xiph.org/releases/ogg/libogg-${OGG_VERSION}.tar.gz && \
    echo ${OGG_SHA256SUM} | sha256sum --check && \
    tar -zx --strip-components=1 -f libogg-${OGG_VERSION}.tar.gz && \
    ./configure --prefix="${PREFIX}" --enable-shared  && \
    make && \
    make install && \
    rm -rf ${DIR}
### libvorbis https://xiph.org/vorbis/
RUN \
    DIR=/tmp/vorbis && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO http://downloads.xiph.org/releases/vorbis/libvorbis-${VORBIS_VERSION}.tar.gz && \
    echo ${VORBIS_SHA256SUM} | sha256sum --check && \
    tar -zx --strip-components=1 -f libvorbis-${VORBIS_VERSION}.tar.gz && \
    ./configure --prefix="${PREFIX}" --with-ogg="${PREFIX}" --enable-shared && \
    make && \
    make install && \
    rm -rf ${DIR}
### libvpx https://www.webmproject.org/code/
RUN \
    DIR=/tmp/vpx && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://codeload.github.com/webmproject/libvpx/tar.gz/v${VPX_VERSION} | \
    tar -zx --strip-components=1 && \
    ./configure --prefix="${PREFIX}" --enable-vp8 --enable-vp9 --enable-vp9-highbitdepth --enable-pic --enable-shared \
    --disable-debug --disable-examples --disable-docs --disable-install-bins  && \
    make && \
    make install && \
    rm -rf ${DIR}
### libmp3lame http://lame.sourceforge.net/
RUN \
    DIR=/tmp/lame && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://sourceforge.net/projects/lame/files/lame/${LAME_VERSION}/lame-${LAME_VERSION}.tar.gz/download | \
    tar -zx --strip-components=1 && \
    ./configure --prefix="${PREFIX}" --bindir="${PREFIX}/bin" --enable-shared --enable-nasm --disable-frontend && \
    make && \
    make install && \
    rm -rf ${DIR}
### fdk-aac https://github.com/mstorsjo/fdk-aac
RUN \
    DIR=/tmp/fdk-aac && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://github.com/mstorsjo/fdk-aac/archive/v${FDKAAC_VERSION}.tar.gz | \
    tar -zx --strip-components=1 && \
    autoreconf -fiv && \
    ./configure --prefix="${PREFIX}" --enable-shared --datadir="${DIR}" && \
    make && \
    make install && \
    rm -rf ${DIR}
## libxcb (and supporting libraries) for screen capture https://xcb.freedesktop.org/
RUN \
    DIR=/tmp/xorg-macros && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://www.x.org/archive//individual/util/util-macros-${XORG_MACROS_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f util-macros-${XORG_MACROS_VERSION}.tar.gz && \
    ./configure --srcdir=${DIR} --prefix="${PREFIX}" && \
    make && \
    make install && \
    rm -rf ${DIR}
RUN \
    DIR=/tmp/xproto && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://www.x.org/archive/individual/proto/xproto-${XPROTO_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f xproto-${XPROTO_VERSION}.tar.gz && \
    ./configure --srcdir=${DIR} --prefix="${PREFIX}" && \
    make && \
    make install && \
    rm -rf ${DIR}
RUN \
    DIR=/tmp/libXau && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://www.x.org/archive/individual/lib/libXau-${XAU_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f libXau-${XAU_VERSION}.tar.gz && \
    ./configure --srcdir=${DIR} --prefix="${PREFIX}" && \
    make && \
    make install && \
    rm -rf ${DIR}
RUN \
    DIR=/tmp/libpthread-stubs && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://xcb.freedesktop.org/dist/libpthread-stubs-${LIBPTHREAD_STUBS_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f libpthread-stubs-${LIBPTHREAD_STUBS_VERSION}.tar.gz && \
    ./configure --prefix="${PREFIX}" && \
    make && \
    make install && \
    rm -rf ${DIR}
RUN \
    DIR=/tmp/libxcb-proto && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://xcb.freedesktop.org/dist/xcb-proto-${XCBPROTO_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f xcb-proto-${XCBPROTO_VERSION}.tar.gz && \
    ACLOCAL_PATH="${PREFIX}/share/aclocal" ./autogen.sh && \
    ./configure --prefix="${PREFIX}" && \
    make && \
    make install && \
    rm -rf ${DIR}
RUN \
    DIR=/tmp/libxcb && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sLO https://xcb.freedesktop.org/dist/libxcb-${LIBXCB_VERSION}.tar.gz && \
    tar -zx --strip-components=1 -f libxcb-${LIBXCB_VERSION}.tar.gz && \
    ACLOCAL_PATH="${PREFIX}/share/aclocal" ./autogen.sh && \
    ./configure --prefix="${PREFIX}" --disable-static --enable-shared && \
    make && \
    make install && \
    rm -rf ${DIR}
## libpng
RUN \
    DIR=/tmp/png && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://git.code.sf.net/p/libpng/code ${DIR} -b v${LIBPNG_VERSION} --depth 1 && \
    ./autogen.sh && \
    ./configure --prefix="${PREFIX}" && \
    make check && \
    make install && \
    rm -rf ${DIR}
## Download ffmpeg https://ffmpeg.org/
RUN  \
    DIR=/tmp/ffmpeg && mkdir -p ${DIR} && cd ${DIR} && \
    curl -sLO https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    tar -jx --strip-components=1 -f ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    ./configure     --disable-debug  --disable-doc    --disable-ffplay   --enable-shared --enable-gpl  --extra-libs=-ldl && \
    make ;  make install

## Build ffmpeg https://ffmpeg.org/
RUN  \
    DIR=/tmp/ffmpeg && cd ${DIR} && \
    ./configure \
    --disable-debug \
    --disable-doc \
    --disable-ffplay \
    --enable-gpl \
    --enable-libfdk_aac \
    --enable-libmp3lame \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libxcb \
    --enable-nonfree \
    --enable-openssl \
    --enable-postproc \
    --enable-shared \
    --enable-small \
    --enable-vaapi \
    --enable-version3 \
    --extra-cflags="-I${PREFIX}/include" \
    --extra-ldflags="-L${PREFIX}/lib" \
    --extra-libs=-ldl \
    --extra-libs=-lpthread \
    --prefix="${PREFIX}" && \
    make clean && \
    make && \
    make install && \
    make distclean && \
    hash -r && \
    cd tools && \
    make qt-faststart && cp qt-faststart ${PREFIX}/bin/

## cleanup
RUN \
    ldd ${PREFIX}/bin/ffmpeg | grep opt/ffmpeg | cut -d ' ' -f 3 | xargs -i cp {} /usr/local/lib/ && \
    for lib in /usr/local/lib/*.so.*; do ln -s "${lib##*/}" "${lib%%.so.*}".so; done && \
    cp ${PREFIX}/bin/* /usr/local/bin/ && \
    cp -r ${PREFIX}/share/ffmpeg /usr/local/share/ && \
    LD_LIBRARY_PATH=/usr/local/lib ffmpeg -buildconf && \
    cp -r ${PREFIX}/include/libav* ${PREFIX}/include/libpostproc ${PREFIX}/include/libsw* /usr/local/include && \
    mkdir -p /usr/local/lib/pkgconfig && \
    for pc in ${PREFIX}/lib/pkgconfig/libav*.pc ${PREFIX}/lib/pkgconfig/libpostproc.pc ${PREFIX}/lib/pkgconfig/libsw*.pc; do \
    sed "s:${PREFIX}:/usr/local:g" <"$pc" >/usr/local/lib/pkgconfig/"${pc##*/}"; \
    done

FROM        base AS release

ENV         LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib64

COPY --from=build /usr/local /usr/local/

RUN \
    apt-get update -y && \
    apt-get install -y --no-install-recommends libva-drm2 libva2 i965-va-driver && \
    rm -rf /var/lib/apt/lists/*