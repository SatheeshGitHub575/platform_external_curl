# Maintainer: Alexey Pavlov <alexpux@gmail.com>
#_variant=-openssl
_variant=-winssl
#_variant=-gnutls

if [ "${_variant}" = "-openssl" ]; then
  _namesuff=
else
  _namesuff="${_variant}"
fi

_realname=curl
pkgbase=mingw-w64-${_realname}
pkgname="${MINGW_PACKAGE_PREFIX}-${_realname}${_namesuff}"
pkgver=7.56.0
pkgrel=2
pkgdesc="Command line tool and library for transferring data with URLs. (mingw-w64)"
arch=('any')
url="https://curl.haxx.se"
license=("MIT")
makedepends=("${MINGW_PACKAGE_PREFIX}-gcc" "${MINGW_PACKAGE_PREFIX}-pkg-config")
depends=("${MINGW_PACKAGE_PREFIX}-gcc-libs"
         "${MINGW_PACKAGE_PREFIX}-c-ares"
         "${MINGW_PACKAGE_PREFIX}-libmetalink"
         "${MINGW_PACKAGE_PREFIX}-libssh2"
         "${MINGW_PACKAGE_PREFIX}-zlib"
         "${MINGW_PACKAGE_PREFIX}-rtmpdump"
         $([[ "$_variant" == "-openssl" ]] && echo \
            "${MINGW_PACKAGE_PREFIX}-ca-certificates" \
            "${MINGW_PACKAGE_PREFIX}-openssl" \
            "${MINGW_PACKAGE_PREFIX}-nghttp2")
         $([[ "$_variant" == "-gnutls" ]] && echo \
            "${MINGW_PACKAGE_PREFIX}-ca-certificates" \
            "${MINGW_PACKAGE_PREFIX}-gnutls")
         )
if [ -n "${_namesuff}" ]; then
  provides=("${MINGW_PACKAGE_PREFIX}-${_realname}")
  conflicts=("${MINGW_PACKAGE_PREFIX}-${_realname}")
fi
options=('staticlibs')
source=("${url}/download/${_realname}-${pkgver}.tar.bz2"{,.asc}
        "0001-Make-cURL-relocatable.patch"
        "0002-cURL-Get-relocatable-base-from-.dll-instead-of-.exe.patch")
sha256sums=('e5b1a92ed3b0c11f149886458fa063419500819f1610c020d62f25b8e4b16cfb'
            'SKIP'
            '577e900086f91adb332f5ddb95adce980c530445d22aa0fbf4b43b25c2efe80e'
            '604b34b5ca9b3520a83a23959f943e330c1ef36b50ee377f8210b59be3e5f62b')
validpgpkeys=('914C533DF9B2ADA2204F586D78E11C6B279D5C91'  # Daniel Stenberg
              '27EDEAF22F3ABCEB50DB9A125CC908FDB71E12C2'
              '4461EAF0F8E9097F48AF0555F9FEAFF9D34A1BDB')

prepare() {
  cd "${_realname}-${pkgver}"
  rm -f lib/pathtools.h lib/pathtools.c > /dev/null 2>&1 || true
  autoreconf -vfi
}

build() {
  local -a extra_config
  if check_option "debug" "y"; then
    extra_config+=( --enable-debug )
  else
    extra_config+=( --disable-debug )
  fi
  mkdir -p "${srcdir}/build-${CARCH}"
  local -a _variant_config
  if [ "${_variant}" = "-winssl" ]; then
    _variant_config+=("--with-ssl")
    _variant_config+=("--without-gnutls")
    _variant_config+=("--with-winssl")
    _variant_config+=('--without-nghttp2')
    _variant_config+=("--without-ca-bundle")
    _variant_config+=("--without-ca-path")  
  elif [ "${_variant}" = "-gnutls" ]; then
    _variant_config+=("--without-ssl")
    _variant_config+=("--with-gnutls")
    _variant_config+=('--without-nghttp2')
    _variant_config+=("--with-ca-bundle=${MINGW_PREFIX}/ssl/certs/ca-bundle.crt")
  elif [ "${_variant}" = "-openssl" ]; then
    _variant_config+=("--without-gnutls")
    _variant_config+=("--with-ssl")
    _variant_config+=("--with-ca-bundle=${MINGW_PREFIX}/ssl/certs/ca-bundle.crt")
    _variant_config+=('--with-nghttp2=${MINGW_PREFIX}/')
  fi
  cd "${srcdir}/build-${CARCH}"
  export curl_disallow_strtok_r="yes"
  ../${_realname}-${pkgver}/configure \
    --prefix=${MINGW_PREFIX} \
    --build=${MINGW_CHOST} \
    --host=${MINGW_CHOST} \
    --target=${MINGW_CHOST} \
    --without-random \
    --without-libssh2 \
    --enable-static \
    --disable-shared \
    --enable-sspi \
    --without-librtmp \
    --without-libidn2 \
    --with-winidn \
    "${_variant_config[@]}" \
    "${extra_config[@]}"
# there's a bug with zsh completion generation script and Windows.
# curl has to specified with the file extension.
  sed -i "s|\/curl > \$\@|\/curl\$\{EXEEXT\} > \$\@|" scripts/Makefile
  make
}

package() {
  cd "${srcdir}/build-${CARCH}"
  make DESTDIR="${pkgdir}" install
  
  local PREFIX_DEPS=$(cygpath -am ${MINGW_PREFIX})
  sed -s "s|${PREFIX_DEPS}|${MINGW_PREFIX}|g" -i ${pkgdir}${MINGW_PREFIX}/bin/curl-config
}