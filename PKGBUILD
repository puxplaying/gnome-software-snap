# Maintainer: pux @forum.manjaro.org
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Yosef Or Boczko <yoseforb@gnome.org>

pkgbase=gnome-software
pkgname=(gnome-software-snap gnome-software-snap-packagekit-plugin)
pkgver=3.36.1
pkgrel=2
pkgdesc="GNOME Software Tools with builtin snap support"
url="https://wiki.gnome.org/Apps/Software/"
arch=(x86_64)
license=(GPL2)
makedepends=(appstream-glib gnome-desktop libpackagekit-glib flatpak fwupd ostree
             docbook-xsl git gobject-introspection gspell gtk-doc meson valgrind
             gnome-online-accounts libxmlb malcontent snapd-glib snapd liboauth)
_commit=56a23c5c716a0d4593c7e790a1b45b678998be28  # tags/3.36.1^0
source=("git+https://gitlab.gnome.org/GNOME/gnome-software.git#commit=$_commit")
sha256sums=('SKIP')

pkgver() {
  cd $pkgbase
  git describe --tags | sed 's/^GNOME_SOFTWARE_//;s/_/./g;s/-/+/g'
}

prepare() {
  cd $pkgbase
}

build() {
  arch-meson $pkgbase build \
    -D snap=true
  ninja -C build
}

check() {
  # build container troubles
  meson test -C build --print-errorlogs || :
}

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_gnome-software-snap() {
  groups=('gnome')
  conflicts=(gnome-software gnome-software-packagekit-plugin)
  provides=("gnome-software=$pkgver" "gnome-software-packagekit-plugin=$pkgver")
  depends=(libxmlb gnome-desktop gsettings-desktop-schemas gspell libpackagekit-glib
           gnome-online-accounts appstream-glib malcontent snapd-glib snapd liboauth)
  optdepends=('flatpak: Flatpak support plugin'
              'fwupd: fwupd support plugin'
              'ostree: OSTree support plugin')

  DESTDIR="$pkgdir" meson install -C build

### Split gnome-software-packagekit-plugin
  _pick packagekit-plugin "$pkgdir"/usr/lib/gs-plugins-*/libgs_plugin_packagekit*.so
  _pick packagekit-plugin "$pkgdir"/usr/lib/gs-plugins-*/libgs_plugin_systemd-updates.so
}

package_gnome-software-snap-packagekit-plugin() {
  pkgdesc="PackageKit support plugin for GNOME Software"
  depends=(archlinux-appstream-data gnome-software packagekit)
  mv packagekit-plugin/* "$pkgdir"
}
