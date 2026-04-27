# Maintainer: Ben Schneider <ben@bens.haus>

pkgname=linux-a3700
pkgver=7.0.1
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
# https://www.kernel.org/pub/linux/kernel/v7.x/sha256sums.asc
sha256sums=('b2c935a36d24980e11e59bed3ca558ea6d67619ec0065faa335cdc0b64d887bf'
            'SKIP'
            '5b905cf60cb13ef2f64785e8e771d2adc0e7c0d412f5e053685f6ea3d236bbc2')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  echo "Setting config..."
  cp ../config .config
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd ${_srcname}
  export KCFLAGS=' -mcpu=cortex-a53'
  export KCPPFLAGS=' -mcpu=cortex-a53'
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

