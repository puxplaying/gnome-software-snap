# Maintainer: Georg Wagner <puxplaying@gmail.com>
# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Yosef Or Boczko <yoseforb@gnome.org>

pkgbase=gnome-software
pkgname=(gnome-software-snap gnome-software-snap-packagekit-plugin)
pkgver=40.3
pkgrel=1
pkgdesc="GNOME Software Tools with builtin snap support"
url="https://wiki.gnome.org/Apps/Software/"
arch=(x86_64)
license=(GPL2)
makedepends=(appstream gsettings-desktop-schemas libpackagekit-glib flatpak
             fwupd docbook-xsl git gobject-introspection gspell gtk-doc meson
             valgrind gnome-online-accounts libxmlb malcontent libhandy snapd-glib snapd)
_commit=615f7dbd2f5eaf98c392f5ec0ee050a65d924ed6  # tags/40.3^0
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
  # Ensure static library is non-LTO compatible
  CFLAGS+=" -ffat-lto-objects"

  arch-meson $pkgbase build \
  -D sysprof=disabled \
  -D snap=true
  meson compile -C build
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
  groups=(gnome)
  conflicts=(gnome-software gnome-software-packagekit-plugin)
  provides=("gnome-software=$pkgver" "gnome-software-packagekit-plugin=$pkgver")
  depends=(libxmlb gsettings-desktop-schemas gspell libpackagekit-glib
           gnome-online-accounts appstream libhandy snapd-glib snapd)
  optdepends=('flatpak: Flatpak support plugin'
              'fwupd: fwupd support plugin'
              'malcontent: Parental control plugin')

  meson install -C build --destdir="$pkgdir"

### Split gnome-software-packagekit-plugin
  local pkglibdir="$pkgdir/usr/lib/gnome-software"
  _pick packagekit-plugin "$pkglibdir"/plugins-*/libgs_plugin_packagekit*.so
  _pick packagekit-plugin "$pkglibdir"/plugins-*/libgs_plugin_systemd-updates.so
}

package_gnome-software-snap-packagekit-plugin() {
  pkgdesc="PackageKit support plugin for GNOME Software"
  depends=(archlinux-appstream-data gnome-software packagekit)
  mv packagekit-plugin/* "$pkgdir"
}
