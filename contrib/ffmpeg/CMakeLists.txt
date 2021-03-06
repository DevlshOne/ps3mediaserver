# author: chocolateboy
# original: https://github.com/pyke369/sffmpeg
# license: GPL
# links:
#
#     build scripts:
#
#         https://github.com/rdp/ffmpeg-windows-build-helpers/blob/master/cross_compile_ffmpeg.sh
#         http://www.slackware.com/~alien/slackbuilds/ffmpeg/build/ffmpeg.SlackBuild
#
#     notes:
#
#         http://betterlogic.com/roger/2011/09/ffmpeg-compileconfigure-windows/
#         http://betterlogic.com/roger/2012/09/libflite-cross-compile-woe/
#         http://ffmpeg.org/trac/ffmpeg/wiki/MacOSXCompilationGuide

PROJECT(ffmpeg)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
INCLUDE(ExternalProject)
INCLUDE_DIRECTORIES(BEFORE ${CMAKE_BINARY_DIR}/include)
LINK_DIRECTORIES(${CMAKE_BINARY_DIR}/lib)

FIND_PACKAGE(PkgConfig REQUIRED)

EXTERNALPROJECT_ADD(
    bzip2
    URL http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
    CONFIGURE_COMMAND ""
    BUILD_COMMAND make PREFIX=${CMAKE_BINARY_DIR} SHARED= XCFLAGS=-I${CMAKE_BINARY_DIR}/include XLDFLAGS=-L${CMAKE_BINARY_DIR}/lib XLIBS=-ldl
    INSTALL_COMMAND make install PREFIX=${CMAKE_BINARY_DIR} SHARED=
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    celt
    URL http://downloads.xiph.org/releases/celt/celt-0.11.1.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    expat
    URL http://downloads.sourceforge.net/project/expat/expat/2.1.0/expat-2.1.0.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    freetype
    DEPENDS bzip2
    URL http://download.savannah.gnu.org/releases/freetype/freetype-2.4.10.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    fontconfig
    DEPENDS expat freetype
    URL http://www.freedesktop.org/software/fontconfig/release/fontconfig-2.10.1.tar.gz
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static --disable-docs
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    frei0r
    URL http://www.piksel.no/frei0r/releases/frei0r-plugins-1.3.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    fribidi
    URL http://fribidi.org/download/fribidi-0.19.4.tar.bz2
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    gsm
    URL http://www.quut.com/gsm/gsm-1.0.13.tar.gz

    # stop the gsm Makefile chmodding its install files to read-only (we want to be able to rebuild/overwrite without interaction)
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/vendor/gsm-permissions.patch

    CONFIGURE_COMMAND ""
    INSTALL_COMMAND make install INSTALL_ROOT=${CMAKE_BINARY_DIR} GSM_INSTALL_INC=${CMAKE_BINARY_DIR}/include chmod=echo
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    libass
    DEPENDS fontconfig freetype fribidi
    URL https://libass.googlecode.com/files/libass-0.10.0.tar.gz
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    mp3lame
    URL http://downloads.sourceforge.net/project/lame/lame/3.99/lame-3.99.5.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    nut
    GIT_REPOSITORY git://git.ffmpeg.org/nut/
    GIT_TAG 5e471c2 # 2011-03-26
    CONFIGURE_COMMAND ""
    BUILD_COMMAND cd src/trunk && make PREFIX=${CMAKE_BINARY_DIR} SHARED= XCFLAGS=-I${CMAKE_BINARY_DIR}/include XLDFLAGS=-L${CMAKE_BINARY_DIR}/lib XLIBS=-ldl
    INSTALL_COMMAND cd src/trunk && make PREFIX=${CMAKE_BINARY_DIR} SHARED= install
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    ogg
    URL http://downloads.xiph.org/releases/ogg/libogg-1.3.0.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

# provides OpenCORE AMR-NB (adaptive multi-rate narrowband) and OpenCORE AMR-WB (wideband)
EXTERNALPROJECT_ADD(
    opencore
    URL http://downloads.sourceforge.net/project/opencore-amr/opencore-amr/opencore-amr-0.1.3.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    openjpeg
    URL https://openjpeg.googlecode.com/files/openjpeg-1.5.1.tar.gz
    CONFIGURE_COMMAND ./bootstrap.sh && ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    openssl
    URL http://www.openssl.org/source/openssl-1.0.1c.tar.gz
    CONFIGURE_COMMAND ./config no-shared no-idea no-mdc2 no-rc5 --prefix=${CMAKE_BINARY_DIR}
    BUILD_COMMAND make depend && make
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    orc
    URL http://code.entropywave.com/download/orc/orc-0.4.16.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_COMMAND make SUBDIRS=orc
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    zlib
    URL http://zlib.net/zlib-1.2.7.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    rtmp
    DEPENDS openssl zlib
    GIT_REPOSITORY git://git.ffmpeg.org/rtmpdump
    GIT_TAG e0056c5 # 2012-07-24
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/vendor/rtmpdump-prefix.patch
    CONFIGURE_COMMAND ""
    BUILD_COMMAND make PREFIX=${CMAKE_BINARY_DIR} SHARED= XCFLAGS=-I${CMAKE_BINARY_DIR}/include XLDFLAGS=-L${CMAKE_BINARY_DIR}/lib XLIBS=-ldl
    INSTALL_COMMAND make PREFIX=${CMAKE_BINARY_DIR} SHARED= install
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    schroedinger
    DEPENDS orc
    URL http://diracvideo.org/download/schroedinger/schroedinger-1.0.11.tar.gz

    # http://www.slackware.com/~alien/slackbuilds/ffmpeg/build/schroedinger_notests.patch
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/vendor/schroedinger-no-tests.patch

    # http://www.slackware.com/~alien/slackbuilds/ffmpeg/build/ffmpeg.SlackBuild
    # "Because of the patch, run autoreconf"
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig autoreconf -vif && PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static

    BUILD_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig make

    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    speex
    DEPENDS ogg
    URL http://downloads.xiph.org/releases/speex/speex-1.2rc1.tar.gz
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-sse  --disable-oggtest --with-ogg=${CMAKE_BINARY_DIR}
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    theora
    DEPENDS ogg
    URL http://downloads.xiph.org/releases/theora/libtheora-1.2.0alpha1.tar.gz
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --disable-oggtest --disable-examples --with-ogg=${CMAKE_BINARY_DIR}
    BUILD_IN_SOURCE 1
)

# http://wiki.multimedia.cx/index.php?title=Ut_Video
EXTERNALPROJECT_ADD(
    utvideo
    URL http://umezawa.dyndns.info/archive/utvideo/utvideo-11.1.1-src.zip
    CONFIGURE_COMMAND ""
    BUILD_COMMAND make
    INSTALL_COMMAND make install prefix=${CMAKE_BINARY_DIR}
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    vo-aacenc
    URL http://sourceforge.net/projects/opencore-amr/files/vo-aacenc/vo-aacenc-0.1.2.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    vo-amrwbenc
    URL http://sourceforge.net/projects/opencore-amr/files/vo-amrwbenc/vo-amrwbenc-0.1.2.tar.gz
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    vorbis
    DEPENDS ogg
    URL http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.2.tar.gz
    CONFIGURE_COMMAND PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --disable-oggtest
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    vpx
    URL http://webm.googlecode.com/files/libvpx-v1.0.0.tar.bz2
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    xavs
    SVN_REPOSITORY https://xavs.svn.sourceforge.net/svnroot/xavs/trunk/
    SVN_REVISION -r55 # 2011-08-21
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    x264
    URL ftp://ftp.videolan.org/pub/x264/snapshots/last_x264.tar.bz2
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/vendor/x264-anonymize.patch
    CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --enable-static
    BUILD_IN_SOURCE 1
)

EXTERNALPROJECT_ADD(
    xvidcore
    URL http://downloads.xvid.org/downloads/xvidcore-1.3.2.tar.gz

    # XXX xvidcore's configure script is alone in not providing a way (e.g. --disable-shared)
    # to prevent it bulding a shared library
    CONFIGURE_COMMAND cd build/generic && ./configure --prefix=${CMAKE_BINARY_DIR} --enable-static

    BUILD_COMMAND cd build/generic && make
    INSTALL_COMMAND cd build/generic && make install
    BUILD_IN_SOURCE 1
)

SET(FFMPEG_DEPENDENCIES
    celt
    frei0r
    gsm
    libass
    mp3lame
    nut
    ogg
    opencore
    openjpeg
    openssl
    rtmp
    schroedinger
    speex
    theora
    utvideo
    vo-aacenc
    vo-amrwbenc
    vorbis
    vpx
    xavs
    x264
    xvidcore
    zlib
)

########################################################################
#
# --extra-cflags=-I/path/to/include/openjpeg-1.5:
#
# ffmpeg's configure script only uses pkg-config for "badly designed
# packages" whose details cannot be obtained in any other way:
# http://lists.mplayerhq.hu/pipermail/ffmpeg-devel/2011-March/108536.html
#
# As a result we have to reinvent pkg-config i.e. provide CFLAGS and/or
# LDFLAGS for the following well-behaved libraries:
#
#     libopenjpeg

########################################################################
#
# disabled options:
#
#     XXX why does the Zeranoe build do this (while stating it supports pthreads)?
#     --disable-pthreads

SET(FFMPEG_CONFIGURE_OPTIONS
    --datadir=${CMAKE_BINARY_DIR}/etc
    --disable-avdevice
    --disable-avisynth # Windows only
    --disable-ffplay
    --disable-ffprobe
    --disable-ffserver
    --enable-bzlib
    --enable-frei0r
    --enable-gpl
    --enable-libass
    --enable-libcelt
    --enable-libfreetype
    --enable-libmp3lame
    --enable-libnut
    --enable-libopencore-amrnb
    --enable-libopencore-amrwb
    --enable-libopenjpeg
    --enable-librtmp
    --enable-libspeex
    --enable-libtheora
    --enable-libutvideo
    --enable-libvo-aacenc
    --enable-libvo-amrwbenc
    --enable-libvorbis
    --enable-libvpx
    --enable-libx264
    --enable-libxavs
    --enable-libxvid
    --enable-runtime-cpudetect
    --enable-static
    --enable-version3
    --enable-zlib
    --extra-cflags=-I${CMAKE_BINARY_DIR}/include
    --extra-cflags=-I${CMAKE_BINARY_DIR}/include/openjpeg-1.5 # see above
    --extra-cflags=-static
    --extra-ldflags=-L${CMAKE_BINARY_DIR}/lib
    --extra-ldflags=-static
    --extra-libs=-ldl
    --prefix=${CMAKE_BINARY_DIR}
)

SET(FFMPEG_CONFIGURE_COMMAND
    env
        "PKG_CONFIG_PATH=${CMAKE_BINARY_DIR}/lib/pkgconfig"

        # http://lists.freedesktop.org/archives/fontconfig/2012-September/004318.html
        # http://lists.freedesktop.org/archives/fontconfig/2012-September/004348.html
        # http://ffmpeg-users.933282.n4.nabble.com/pkg-config-static-td4653220.html
        "pkg_config=${PKG_CONFIG_EXECUTABLE} --static"

    ./configure ${FFMPEG_CONFIGURE_OPTIONS}
)

EXTERNALPROJECT_ADD(
    ffmpeg
    DEPENDS ${FFMPEG_DEPENDENCIES}
    URL http://ffmpeg.org/releases/ffmpeg-1.0.tar.gz
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/vendor/ffmpeg-banner.patch
    CONFIGURE_COMMAND ${FFMPEG_CONFIGURE_COMMAND}
    BUILD_IN_SOURCE 1
)
