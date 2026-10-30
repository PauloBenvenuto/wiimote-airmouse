# Paquete local: se construye desde esta carpeta (makepkg -si), sin descargar nada.
# La versión sale de git: r<commits>.<hash>.
pkgname=wiimote-airmouse
pkgver=r0
pkgrel=1
pkgdesc="Wii Remote with MotionPlus as a gyro air mouse for Hyprland (evdev to uinput)"
arch=(any)
license=(GPL-3.0-or-later)
depends=(python python-evdev python-pywayland wayland-protocols bluez-utils libnotify)
optdepends=('quickshell: on-screen keyboard and workspace drag UI ([ui] ipc)')
makedepends=(git)
install=wiimote-airmouse.install

pkgver() {
    cd "$startdir"
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

package() {
    cd "$startdir"
    install -Dm755 wiimote-airmouse wii-keyboard -t "$pkgdir/usr/bin"
    install -Dm644 protocols/input-method-unstable-v2.xml -t "$pkgdir/usr/share/$pkgname/protocols"
    install -Dm644 airmouse.example.toml README.md -t "$pkgdir/usr/share/doc/$pkgname"
    install -Dm644 LICENSE -t "$pkgdir/usr/share/licenses/$pkgname"
    install -Dm644 wiimote-airmouse.service -t "$pkgdir/usr/lib/systemd/user"
    install -Dm644 70-wiimote-airmouse.rules -t "$pkgdir/usr/lib/udev/rules.d"
}
