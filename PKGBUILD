# Maintainer: Georg Wagner <puxplaying@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Yosef Or Boczko <yoseforb@gnome.org>

pkgbase=gnome-software
pkgname=(gnome-software-snap gnome-software-snap-packagekit-plugin)
pkgver=40.rc+11+g37f98cc0
pkgrel=1
pkgdesc="GNOME Software Tools with builtin snap support"
url="https://wiki.gnome.org/Apps/Software/"
arch=(x86_64)
license=(GPL2)
makedepends=(appstream appstream-glib gnome-desktop libpackagekit-glib flatpak fwupd 
             docbook-xsl git gobject-introspection gspell gtk-doc meson valgrind
             gnome-online-accounts libxmlb malcontent sysprof snapd-glib snapd liboauth cmake libhandy)
_commit=37f98cc0ef1e384db18a3acc077597ee9c1ea9b3  # tags/40.rc^0
source=("git+https://gitlab.gnome.org/GNOME/gnome-software.git#commit=$_commit"
        'git+https://gitlab.gnome.org/GNOME/libhandy.git')
sha256sums=('SKIP'
            'SKIP')

pkgver() {
  cd $pkgbase
  git describe --tags | sed 's/^GNOME_SOFTWARE_//;s/_/./g;s/-/+/g'
}

prepare() {
  cd $pkgbase
  git submodule init
  git submodule set-url subprojects/libhandy "$srcdir/libhandy"
  git submodule update
}

build() {
  # Ensure static library is non-LTO compatible
  CFLAGS+=" -ffat-lto-objects"

  arch-meson $pkgbase build \
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
  depends=(libxmlb gnome-desktop gsettings-desktop-schemas gspell libpackagekit-glib
           gnome-online-accounts appstream-glib snapd-glib snapd liboauth)
  optdepends=('flatpak: Flatpak support plugin'
              'fwupd: fwupd support plugin'
              'malcontent: Parental control plugin')

  DESTDIR="$pkgdir" meson install -C build

### Split gnome-software-packagekit-plugin
  _pick packagekit-plugin "$pkgdir"/usr/lib/gnome-software/plugins-*/libgs_plugin_packagekit*.so
  _pick packagekit-plugin "$pkgdir"/usr/lib/gnome-software/plugins-*/libgs_plugin_systemd-updates.so
}

package_gnome-software-snap-packagekit-plugin() {
  pkgdesc="PackageKit support plugin for GNOME Software"
  depends=(archlinux-appstream-data gnome-software packagekit)
  mv packagekit-plugin/* "$pkgdir"
}
