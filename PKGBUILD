_pkgbase=nss
pkgbase=$_pkgbase-pure
pkgname=(nss-pure ca-certificates-mozilla-pure)
pkgver=3.19.1
pkgrel=1
pkgdesc="Mozilla Network Security Services (with no patches)"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/security/pki/nss/"
license=('MPL' 'GPL')
_nsprver=4.10.8
depends=("nspr>=${_nsprver}" 'sqlite' 'zlib' 'sh' 'p11-kit')
makedepends=('perl' 'python2')
options=('!strip' '!makeflags' 'staticlibs')
source=("https://ftp.mozilla.org/pub/mozilla.org/security/nss/releases/NSS_${pkgver//./_}_RTM/src/${_pkgbase}-${pkgver}.tar.gz"
        certdata2pem.py
        bundle.sh
        nss.pc.in
        nss-config.in)
sha256sums=('b7be709551ec13206d8e3e8c065b894fa981c11573115e9478fa051029c52fff'
            'af13c30801a8a27623948206458432a4cf98061b75ff6e5b5e03912f93c034ee'
            '045f520403f715a4cc7f3607b4e2c9bcc88fee5bce58d462fddaa2fdb0e4c180'
            'b9f1428ca2305bf30b109507ff335fa00bce5a7ce0434b50acd26ad7c47dd5bd'
            'e44ac5095b4d88f24ec7b2e6a9f1581560bd3ad41a3d198596d67ef22f67adb9')

prepare() {
  mkdir certs

  cd nss-$pkgver

  ln -sr nss/lib/ckfw/builtins/certdata.txt ../certs/
  ln -sr nss/lib/ckfw/builtins/nssckbi.h ../certs/
}


build() {
  cd certs
  python2 ../certdata2pem.py

  cd ..
  sh bundle.sh

  cd nss-$pkgver/nss
  export BUILD_OPT=1
  export NSS_USE_SYSTEM_SQLITE=1
  export NSS_ENABLE_ECC=1
  export NSPR_INCLUDE_DIR="`nspr-config --includedir`"
  export NSPR_LIB_DIR="`nspr-config --libdir`"
  export XCFLAGS="${CFLAGS}"

  [ "$CARCH" = "x86_64" ] && export USE_64=1

  make -C coreconf
  make -C lib/dbm
  make
}

package_nss-pure() {
  provides=(nss)
  conflicts=(nss)
  install=nss.install

  cd nss-$pkgver
  install -d "$pkgdir"/usr/{bin,include/nss,lib/pkgconfig}

  NSS_VMAJOR=$(grep '#define.*NSS_VMAJOR' nss/lib/nss/nss.h | awk '{print $3}')
  NSS_VMINOR=$(grep '#define.*NSS_VMINOR' nss/lib/nss/nss.h | awk '{print $3}')
  NSS_VPATCH=$(grep '#define.*NSS_VPATCH' nss/lib/nss/nss.h | awk '{print $3}')

  sed ../nss.pc.in \
    -e "s,%libdir%,/usr/lib,g" \
    -e "s,%prefix%,/usr,g" \
    -e "s,%exec_prefix%,/usr/bin,g" \
    -e "s,%includedir%,/usr/include/nss,g" \
    -e "s,%NSPR_VERSION%,${_nsprver},g" \
    -e "s,%NSS_VERSION%,${pkgver},g" \
    > "$pkgdir/usr/lib/pkgconfig/nss.pc"
  ln -s nss.pc "$pkgdir/usr/lib/pkgconfig/mozilla-nss.pc"

  sed ../nss-config.in \
    -e "s,@libdir@,/usr/lib,g" \
    -e "s,@prefix@,/usr/bin,g" \
    -e "s,@exec_prefix@,/usr/bin,g" \
    -e "s,@includedir@,/usr/include/nss,g" \
    -e "s,@MOD_MAJOR_VERSION@,${NSS_VMAJOR},g" \
    -e "s,@MOD_MINOR_VERSION@,${NSS_VMINOR},g" \
    -e "s,@MOD_PATCH_VERSION@,${NSS_VPATCH},g" \
    > "$pkgdir/usr/bin/nss-config"
  chmod 755 "$pkgdir/usr/bin/nss-config"

  cd dist/*.OBJ/bin
  install -t "$pkgdir/usr/bin" *util shlibsign signtool signver ssltap

  cd ../lib
  install -t "$pkgdir/usr/lib" *.so
  install -t "$pkgdir/usr/lib" -m644 libcrmf.a *.chk

  cd ../../public/nss
  install -t "$pkgdir/usr/include/nss" -m644 *.h

  rm "$pkgdir/usr/lib/libnssckbi.so"
  ln -s libnssckbi-p11-kit.so "$pkgdir/usr/lib/libnssckbi.so"
}

package_ca-certificates-mozilla-pure() {
  pkgdesc="Mozilla's set of trusted CA certificates"
  provides=(ca-certificates-mozilla)
  conflicts=(ca-certificates-mozilla)
  depends=(ca-certificates-utils)
  install=ca-certificates-mozilla.install

  local _certdir="$pkgdir/usr/share/ca-certificates/trust-source"
  install -Dm644 ca-bundle.trust.crt "$_certdir/mozilla.trust.crt"
  install -Dm644 ca-bundle.neutral-trust.crt "$_certdir/mozilla.neutral-trust.crt"
  install -Dm644 ca-bundle.supplement.p11-kit "$_certdir/mozilla.supplement.p11-kit"
}
