pkgname=rpi-libcamera-apps-git
_pkgname=rpi-libcamera-apps
_appname=libcamera-apps
pkgver=r118.9666fa7
pkgrel=1
pkgdesc='This is a small suite of libcamera-based apps for archlinux like AARCH64 that aim to copy the functionality of the existing "raspicam" apps.'
arch=('aarch64')
url='https://github.com/raspberrypi/libcamera-apps'
makedepends=(
    "gcc"
    "git"
    "make"
    "cmake"
    "libepoxy"
    "boost-libs"
    "libdrm"
    "libexif"
)
optdepends=(
    "libcamera-git"
    "gstreamer"
    "gstreamermm"
)
license=('BSD')
options=('!buildflags')
source=('git://github.com/raspberrypi/libcamera-apps.git/')
md5sums=('SKIP')
provides=("$_appname")
conflicts=("$_appname")

pkgver() {
    cd "$_appname"
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
    mkdir -p "$srcdir/$_appname/build"
}

build() {
    cd "$srcdir/$_appname/build"
    cmake -DCMAKE_INSTALL_PREFIX=/usr ..
    make -j4
}

package() {
    cd "$srcdir/$_appname/build"
    make DESTDIR="$pkgdir" install
    install -d -m 755 "$pkgdir"/usr/share/licenses/"$_appname"
    install -D -m 644 "$srcdir/$_appname/license.txt"  "$pkgdir"/usr/share/licenses/"$_appname"/LICENSE
}
