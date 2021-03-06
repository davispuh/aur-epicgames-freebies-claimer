# Maintainer: Dāvis Mosāns <davispuh at gmail dot com>

pkgname=epicgames-freebies-claimer
pkgver=V1.4.1
pkgrel=1
pkgdesc="Automatically claim free game promotions from the Epic Game Store."
arch=("any")
url="https://github.com/Revadike/epicgames-freebies-claimer"
license=("MIT")
depends=("nodejs" "sh")
makedepends=("npm" "sed" "curl" "jq")
source=("epicgames-freebies-claimer" "epicgames-freebies-claimer.service")
sha256sums=("ef1f2943deb5776fa3fa44b850c202373b222b8232c02832f102c80f693d0630"
            "26001be311424a616f2c4ca35de68d5427d918b4849857e36dba0eeb88ac6a22")

pkgver() {
    curl -s https://api.github.com/repos/Revadike/epicgames-freebies-claimer/releases/latest | jq -r '.tag_name'
}

prepare() {
    tarballUrl=$(curl -s https://api.github.com/repos/Revadike/epicgames-freebies-claimer/releases/latest | jq -r '.tarball_url')
    curl -L "$tarballUrl" > "$srcdir/tarball.tar.gz"
}

package() {
    npm install -g --user root --cache "$srcdir/npm-cache" --prefix "$pkgdir/usr" "$srcdir/tarball.tar.gz"

    # Non-deterministic race in npm gives 777 permissions to random directories.
    # See https://github.com/npm/npm/issues/9359 for details.
    find "$pkgdir/usr" -type d -exec chmod 755 {} +

    find "$pkgdir" -name package.json -print0 | xargs -r0 sed -i "s|$pkgdir||"
    find "$pkgdir" -name package.json -print0 | xargs -r0 sed -i "s|$srcdir|/usr/lib/node_modules/epicgames-freebies-claimer|"

    sed -i "s|require(\`\${__dirname}/config.json\`)|require(\`\${process.env.HOME}/.config/epicgames/config.json\`)|" "$pkgdir/usr/lib/node_modules/epicgames-freebies-claimer/claimer.js"

    install -Dm755 "$srcdir/epicgames-freebies-claimer" -t "$pkgdir/usr/bin/"

    entry=$(cat "$pkgdir/usr/lib/node_modules/epicgames-freebies-claimer/package.json" | jq ".main")
    sed -i "s|entry=.*|entry=$entry|" "$pkgdir/usr/bin/epicgames-freebies-claimer"

    install -Dm644 "$srcdir/epicgames-freebies-claimer.service" -t "$pkgdir/usr/lib/systemd/user/"
}

