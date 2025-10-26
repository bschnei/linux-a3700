# Maintainer: Ben Schneider <ben@bens.haus>

pkgname=linux-a3700
pkgver=6.17.5
pkgrel=1
pkgdesc='Kernel and modules for Marvell Armada A3700 SoC'
arch=(aarch64)
url='https://www.kernel.org/'
license=('GPL-2.0 WITH Linux-syscall-note')
makedepends=(
  bc
  tar
  xz
)
options=(
  !debug
  !strip
)
# make -s kernelrelease produces a version number with a trailing .0
# on new versions (e.g. 6.14 => 6.14.0) but the trailing .0 is not used
# to name the source .tar.xz files.
if [ ${pkgver: -2} = ".0" ]; then
  _srcname=linux-${pkgver:0:-2}
else
  _srcname=linux-${pkgver}
fi
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  config
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('c05faf36e9c2164be723cf6ada8533788804d48f9dd2fe1be2ccee3616a92bce'
            'SKIP'
            'b50106d5093891d6295fb69c0b2e6ae8d4807a6876546e87c10c43cd314efd46')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ../config .config
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd ${_srcname}
  make vmlinuz.efi modules
}

package() {
  pkgdesc="$pkgdesc"
  depends=(
    coreutils
    kmod
  )
  optdepends=(
    'initramfs: initial ramdisk creation'
    'linux-firmware-marvell: wifi and bluetooth driver (mwifiex)'
    'wireless-regdb: to set the correct wireless channels of your country'
  )
  provides=(WIREGUARD-MODULE)

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  echo "Installing modules..."
  ZSTD_CLEVEL=19 make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  # remove build link
  rm "$modulesdir"/build
}

# vim:set ts=8 sts=2 sw=2 et:

