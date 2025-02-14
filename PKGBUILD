# Maintainer: Faruk Dikcizgi <boogiepop@gmx.de>
# Contributor: Jan Keith Darunday <jkcdarunday+aur.archlinux.org@gmail.com>
# Contributor: Laurent Carlier <lordheavym@gmail.com>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Contributor: Dan Johansen <strit@manjaro.org>

# ALARM: Kevin Mihelich <kevin@archlinuxarm.org>
#  - Disabled all artifacts except panfrost since they cause color issues on runtime 
#  - Removed Gallium3D drivers/packages for chipsets that don't exist in our ARM devices (intel, VMware svga).
#  - added broadcom and panfrost vulkan packages
#  - enable lto for aarch64

pkgname=mesa-panfork-git
pkgdesc="Libgl & gbm with panfrost for Mali G610 from icecream95"
pkgver=r164484.120202c6757
pkgrel=2
arch=('aarch64' 'armv7h')
makedepends=('meson' 'python-mako' 'python-packaging' 'bison' 'flex' 'cmake'
              'libxfixes' 'libxxf86vm' 'lm_sensors' 'libxshmfence' 'wayland-protocols' 'libdrm' 'libxrandr' 'libglvnd')
url="https://gitlab.com/hbiyik/mesa"
license=('custom')
options=('!lto')
source=("mesa::git+https://github.com/hbiyik/mesa.git#branch=panfork"
        LICENSE)
sha512sums=('SKIP'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7')

validpgpkeys=('8703B6700E7EE06D7A39B8D6EDAE37B02CEB490D'  # Emil Velikov <emil.l.velikov@gmail.com>
              '946D09B5E4C9845E63075FF1D961C596A7203456'  # Andres Gomez <tanty@igalia.com>
              'E3E8F480C52ADD73B278EE78E1ECBE07D7D70895'  # Juan Antonio Suárez Romero (Igalia, S.L.) <jasuarez@igalia.com>
              'A5CC9FEC93F2F837CB044912336909B6B25FADFA'  # Juan A. Suarez Romero <jasuarez@igalia.com>
              '71C4B75620BC75708B4BDB254C95FAAB3EB073EC'  # Dylan Baker <dylan@pnwbakers.com>
              '57551DE15B968F6341C248F68D8E31AFC32428A6') # Eric Engestrom <eric@engestrom.ch>

pkgver() {
  cd mesa
  local _ver
  _ver=$(<VERSION)
  _ver=${_ver//-/.}
  printf "%s.r%s.%s" $_ver "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

build() {
  arch-meson --reconfigure mesa build \
    -Dgallium-drivers=panfrost \
    -Dvulkan-drivers= \
    -Ddri3=enabled \
    -Degl=enabled \
    -Dllvm=disabled \
    -Dmicrosoft-clc=disabled \
    -Dintel-clc=disabled \
    -Dplatforms=wayland,x11 \
    -Dgallium-nine=true \
    -Dgbm=enabled \
    -Dgles1=disabled \
    -Dgles2=enabled \
    -Dglx=dri \
    -Dshared-glapi=enabled \
    -Dglvnd=true

  ninja -C build
  meson compile -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="${srcdir}/fakeinstall" meson install -C build
}

package() {
  depends=('egl-wayland' 'libxfixes' 'libxxf86vm' 'lm_sensors' 'libxshmfence' 'libdrm' 'libxrandr' 'libglvnd')
  provides=('mesa-libgl' 'opengl-driver' 'mesa')
  conflicts=('mesa-libgl' 'mesa')

  #install libs
  _install fakeinstall/usr/share/drirc.d/00-mesa-defaults.conf
  _install fakeinstall/usr/lib/dri/*_dri.so
  _install fakeinstall/usr/lib/lib{gbm,glapi}.so*
  _install fakeinstall/usr/lib/libGL*
  _install fakeinstall/usr/lib/libEG*
  _install fakeinstall/usr/include
  _install fakeinstall/usr/lib/pkgconfig

  _install fakeinstall/usr/share/glvnd/egl_vendor.d/50_mesa.json
  _install fakeinstall/usr/lib/d3d
  
  find fakeinstall -depth -print0 | xargs -0 rmdir

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    mv -v "${src}" "${dir}/"
  done
}