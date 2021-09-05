# AArch64 multi-platform
# Contributor: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: Dan Johansen <strit@manjaro.org>

pkgbase=linux-pbp
_srcname=linux
_kernelname=${pkgbase#linux}
_desc="Linux kernel with patches for the Pinebook Pro."
pkgver=5.14.1
pkgrel=1
arch=('aarch64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'dtc')
options=('!strip')

_patches=(
	'0001-soc-rockchip-Add-rockchip-suspend-mode-driver.patch'
	'0002-firmware-Add-Rockchip-SIP-driver.patch'
	'0003-tty-serdev-support-shutdown-op.patch'
	'0004-bluetooth-hci_serdev-Clear-registered-bit-on-unregis.patch'
	'0005-bluetooth-hci_bcm-disable-power-on-shutdown.patch'
	'0006-mmc-core-pwrseq_simple-disable-mmc-power-on-shutdown.patch'
	'0007-regulator-core-add-generic-suspend-states-support.patch'
	'0008-sound-soc-codecs-es8316-Run-micdetect-only-if-jack-s.patch'
	'0009-ASoC-soc-jack.c-supported-inverted-jack-detect-GPIOs.patch'
	'0010-arm64-dts-rockchip-add-default-rk3399-rockchip-suspe.patch'
	'0011-arm64-dts-rockchip-enable-earlycon.patch'
	'0012-arm64-dts-rockchip-reserve-memory-for-ATF-rockchip-S.patch'
	'0013-arm64-dts-rockchip-use-power-led-for-disk-activity-i.patch'
	'0014-arm64-dts-rockchip-add-oficially-unsupported-2GHz-op.patch'
	'0015-rm64-dts-rockchip-add-rockchip-suspend-node.patch'
	'0016-arm64-configs-add-defconfig-for-Pinebook-Pro.patch'
	'0017-soc-rockchip-Port-rockchip_pm_config-driver-to-Linux.patch'
	'0018-arm64-dts-rockchip-remove-cdn-dp.patch'
)

source=("git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git#tag=v${pkgver}"
        'config'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook'
        "${_patches[@]}"
    )
md5sums=('SKIP'
         'adee4e7a6245dde95177e879344d890d'
         '86d4a35722b5410e3b29fc92dae15d4b'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         '3dc88030a8f2f5a5f97266d99b149f77'
         '4d163f49bcf294889d6b6db636e5be38'
         'b19cfd6998d3bcf09a9eb4c4812215a7'
         '80402f1550ae4c809f3e18b80498cb9f'
         '04855fe513563128d8fa81cad4187208'
         'f7bc3a6100af698bbe51938f60067fee'
         '531c1403f8c02d6e3ade9f5b3f93139a'
         'dfe9345b526683d83d4582dcdf23bd3a'
         '1940c1c0e4b4713d1e8cdb11fb672311'
         '038ac1789e4bada763cf1733357468da'
         'eb58c80651ab527de4bf8a0c7d4a333d'
         '988b1a32a374f1485333d438da41ba9c'
         '4b690d5071f03784ff113d0fcdb53388'
         '3fde3c2df2badefc5595a980f007ef72'
         '7ddbbb9ae78d6f4f5196fa1bc495a8da'
         '2d5f25d6e2839daa9abe4ce89daeb665'
         'a0e8d15a6e0da5e35d66e3df6672a2e3'
         'c43edbc576b0739b1c089e05d3904fa5'
         '8b15a5d6bbeee59678eb2c13cc77e459')

prepare() {
  cd ${_srcname}

  for patch in "${_patches[@]}"; do
    git apply "${srcdir}/$patch"
  done

  cat "${srcdir}/config" > ./.config

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh
}

build() {
  cd ${_srcname}

  make "${MAKEFLAGS[@]}" prepare

  #make menuconfig
  #exit 1

  unset LDFLAGS
  make "${MAKEFLAGS[@]}" Image Image.gz modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make "${MAKEFLAGS[@]}" DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  replaces=('linux-armv8')
  conflicts=('linux')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=${pkgname}.install

  cd ${_srcname}

  KARCH=arm64

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install
  cp arch/$KARCH/boot/Image{,.gz} "${pkgdir}/boot"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # add vmlinux
  install -Dt "${pkgdir}/usr/lib/modules/${_kernver}/build" -m644 vmlinux

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd ${_srcname}
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  cp -t "${_builddir}" -a include scripts

  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile
  install -Dt "${_builddir}/arch/${KARCH}/kernel" -m644 arch/${KARCH}/kernel/asm-offsets.s #arch/$KARCH/kernel/module.lds

  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include
  mkdir -p "${_builddir}/arch/arm"
  cp -t "${_builddir}/arch/arm" -a arch/arm/include

  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # add xfs and shmem for aufs building
  mkdir -p "${_builddir}"/{fs/xfs,mm}

  # copy in Kconfig files
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  # remove unneeded architectures
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ || ${_arch} == */arm/ ]] && continue
    rm -r "${_arch}"
  done

  # remove files already in linux-docs package
  rm -r "${_builddir}/Documentation"

  # remove now broken symlinks
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  # Fix permissions
  chmod -R u=rwX,go=rX "${_builddir}"

  # strip scripts directory
  local _binary _strip
  while read -rd '' _binary; do
    case "$(file -bi "${_binary}")" in
      *application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
      *application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
      *application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
      *) continue ;;
    esac
    /usr/bin/strip ${_strip} "${_binary}"
  done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
