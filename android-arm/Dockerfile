FROM dmartinpro/cross-compiler:android-arm
MAINTAINER David Martin "dmartinpro@gmail.com"

RUN apt-get update && apt-get install -y automake libpcre3-dev bison yodl wget

RUN mkdir -p /build
WORKDIR /build

# Install Boost.System
ENV BOOST_VERSION 1.60.0
RUN curl -L http://sourceforge.net/projects/boost/files/boost/${BOOST_VERSION}/boost_`echo ${BOOST_VERSION} | sed 's/\\./_/g'`.tar.bz2/download | tar xvj && \
    cd boost_`echo ${BOOST_VERSION} | sed 's/\\./_/g'`/ && \
    ./bootstrap.sh --prefix=${CROSS_ROOT} && \
    echo "using gcc : android : ${CROSS_TRIPLE}-c++ ;" > ${HOME}/user-config.jam && \
    ./b2 --with-date_time --with-system --prefix=${CROSS_ROOT} toolset=gcc-android link=static variant=release threading=multi target-os=linux install && \
    rm -rf ${HOME}/user-config.jam && \
    rm -rf `pwd`

# Install OpenSSL
ENV OPENSSL_VERSION 1.0.2e
RUN curl -L http://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz | tar xvz && \
    cd openssl-${OPENSSL_VERSION}/ && \
    CROSS_COMPILE=${CROSS_TRIPLE}- ./Configure threads no-shared linux-armv4 --prefix=${CROSS_ROOT} && \
    make && make install && \
    rm -rf `pwd`

# Install libtorrent
ENV LIBTORRENT_VERSION 1.0.7
RUN wget -nc https://github.com/arvidn/libtorrent/releases/download/libtorrent-`echo ${LIBTORRENT_VERSION} | sed 's/\\./_/g'`/libtorrent-rasterbar-${LIBTORRENT_VERSION}.tar.gz && \
    tar xf libtorrent-rasterbar-${LIBTORRENT_VERSION}.tar.gz && \
    cd libtorrent-rasterbar-${LIBTORRENT_VERSION}/ && \
    sed -i 's/$PKG_CONFIG openssl --libs-only-/$PKG_CONFIG openssl --static --libs-only-/' ./configure && \
    sed -i -e s/Windows.h/windows.h/ -e s/Wincrypt.h/wincrypt.h/ ./ed25519/src/seed.cpp && \
    \
    PKG_CONFIG_PATH=${CROSS_ROOT}/lib/pkgconfig/ \
    CC=${CROSS_TRIPLE}-gcc CXX=${CROSS_TRIPLE}-g++ \
    CFLAGS="${CFLAGS} -O2 -DINT64_MAX=0x7fffffffffffffffLL -DINT16_MAX=32767 -DINT16_MIN=-32768" \
    CXXFLAGS="${CXXFLAGS} ${CFLAGS}" \
    ./configure --enable-static --disable-shared --disable-pool-allocators --disable-deprecated-functions \
    --host=${CROSS_TRIPLE} --prefix=${CROSS_ROOT} \
    --with-boost=${CROSS_ROOT} --with-boost-libdir=${CROSS_ROOT}/lib && \
    \
    make && make install && \
    rm -rf `pwd`

# Install SWIG
# Need to build from >3.0.2 because Go 1.4.2
ENV SWIG_VERSION rel-3.0.4
RUN curl -L https://github.com/swig/swig/archive/${SWIG_VERSION}.tar.gz | tar xvz && \
    cd swig-${SWIG_VERSION}/ && \
    curl -L https://github.com/steeve/swig/compare/swig:${SWIG_VERSION}...packed-struct-libtorrent.diff | patch -p1 && \
    ./autogen.sh && \
    ./configure && make && make install && \
    rm -rf `pwd`

# Install Golang
# Patch in order to avoid a duplicate definition of PTHREAD_KEYS_MAX => Seems patched in Go 1.5
ENV GO_VERSION 1.4.3
RUN cd /usr/local && \
    curl -L http://golang.org/dl/go${GO_VERSION}.src.tar.gz | tar xvz
RUN cd /usr/local/go/src/runtime/cgo/ && \
    sed -e 's/#define PTHREAD_KEYS_MAX 128/#ifndef PTHREAD_KEYS_MAX\n#define PTHREAD_KEYS_MAX 128\n#endif/g' gcc_android_arm.c > gcc_android_arm.c2 && mv gcc_android_arm.c2 gcc_android_arm.c
RUN cd /usr/local/go/src/ && CC_FOR_TARGET=${CROSS_TRIPLE}-cc CXX_FOR_TARGET=${CROSS_TRIPLE}-c++ GOOS=android GOARCH=arm GOARM=7 CGO_ENABLED=1 ./make.bash
ENV PATH ${PATH}:/usr/local/go/bin

WORKDIR /
RUN rm -rf /build
