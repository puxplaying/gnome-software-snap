# Maintainer: Georg Wagner <puxplaying@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Yosef Or Boczko <yoseforb@gnome.org>

pkgbase=gnome-software
pkgname=(gnome-software-snap gnome-software-snap-packagekit-plugin)
pkgver=40.beta
pkgrel=1
pkgdesc="GNOME Software Tools with builtin snap support"
url="https://wiki.gnome.org/Apps/Software/"
arch=(x86_64)
license=(GPL2)
makedepends=(appstream appstream-glib gnome-desktop libpackagekit-glib flatpak fwupd 
             docbook-xsl git gobject-introspection gspell gtk-doc meson valgrind
             gnome-online-accounts libxmlb malcontent sysprof snapd-glib snapd liboauth)
_commit=4f7915fa6db4cacd93a665a9c5ef7164663e8c59  # tags/3.38.1^0
source=("git+https://gitlab.gnome.org/GNOME/gnome-software.git#commit=$_commit"
        'git+https://gitlab.gnome.org/GNOME/libhandy.git'
        '0001-enable-snapd-support.patch')
sha256sums=('SKIP'
            'SKIP'
            '690862103c52510f49d241a9624aa168b39cb8512ce77b7f746dce9e96d796c4')

pkgver() {
  cd $pkgbase
  git describe --tags | sed 's/^GNOME_SOFTWARE_//;s/_/./g;s/-/+/g'
}

prepare() {
  cd $pkgbase
  git submodule init
  git submodule set-url subprojects/libhandy "$srcdir/libhandy"
  git submodule update
  patch -p1 -i "${srcdir}/0001-enable-snapd-support.patch"
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
  _pick packagekit-plugin "$pkgdir"/usr/lib/gs-plugins-*/libgs_plugin_packagekit*.so
  _pick packagekit-plugin "$pkgdir"/usr/lib/gs-plugins-*/libgs_plugin_systemd-updates.so
}

package_gnome-software-snap-packagekit-plugin() {
  pkgdesc="PackageKit support plugin for GNOME Software"
  depends=(archlinux-appstream-data gnome-software packagekit)
  mv packagekit-plugin/* "$pkgdir"
}
