pkgname=init-nitro
pkgver=0.3.1
_nrev="d4f6b1a31962ce95febcd22ade9f0cafeaa625c1"
_rcrev="e0c4e306e448d2ac7a8a314133995ee37bc48f92a"
_rver=2.2.0
_rpver=20250506
pkgrel=1
pkgdesc='simple init'
arch=('x86_64' 'aarch64')
url='https://github.com/leahneukirchen/nitro'
license=('GPL')
#provides=('runit')
depends=('glibc' 'bash')
conflicts=('runit' 'runit-rc' 'svc-manager' 'eudev-runit')
makedepends=()
optdepends=()
groups=()
backup=()
source=(
		"nitro-$pkgver-${_nrev:0:10}.tar.gz::https://github.com/leahneukirchen/nitro/archive/${_nrev}.tar.gz"
		"git+https://gitea.artixlinux.org/artix/runit-rc.git"
		"http://smarden.org/runit/runit-${_rver}.tar.gz"
		"runit-patches-${_rpver}.tar.xz::https://github.com/clan/runit/releases/download/runit-${_rver}-r2/runit-${_rver}-patches-${_rpver}.tar.xz"
        "runit-artix-20210904.tar.gz::https://gitea.artixlinux.org/artix/runit-artix/archive/20210904.tar.gz"
		"etc-nitro.tar.xz"
		"000-services.patch"
		"001-runit-rc.patch"
		"shutdown"
		"nsm"
		 )
sha256sums=('6ee759a4cab083c67e59662e88234a3a68f447a9cc81058156e66edd1dc47349'
            'SKIP'
            '95ef4d2868b978c7179fe47901e5c578e11cf273d292bd6208bd3a7ccb029290'
            'bbd115a9612c5a8df932cd43c406393538389b248ad44f1d9903bc0e2850e173'
            '191f7d0ad00183ab3a8820ca9bf4295de8af6d9bfa06571653e8fd3d8280e63a'
            '855e06a4faf72c096d8ffc988c17b2579ac3ba7f01c5f9e0b96df06768406cd0'
            'a1f473abe28b060bef7dde303577d0b5c1b373caf3069f93af6bd2c8ea114353'
            '594819bda53593ac4ffbdc12e022786609177527d89d34e8675093a884e68a9a'
            '07aecac5688b90e9ba4c0b169175fc8d359a393b7f011ae39cac570242bdb906'
            '15d78a4cb51e9567142a71441e4501c8d08fdd9c17f875c75a2b4f7c54cbcd19')
validpgpkeys=()

prepare() {
	local x
	ln -sf nitro-${_nrev} $pkgname-$pkgver
	cd $pkgname-$pkgver
	patch -Np1 -i "$srcdir"/000-services.patch
	cd "$srcdir/runit-rc"
	patch -Np1 -i "$srcdir"/001-runit-rc.patch

	cd "$srcdir/admin/runit-${_rver}/src"
	for x in "${srcdir}"/patches/*; do
		patch -Np2 -i "$x"
	done
}

build() {
	cd $pkgname-$pkgver
	make
	cd "${srcdir}/runit-rc"
	make
	#from runit pkg
	#cc ${CFLAGS} halt.c -o halt ${LDFLAGS}

	#cd ${_pkgname}
	#make SERVICEDIR="${_servicedir}"

	cd "${srcdir}/admin/runit-${_rver}"

	CFLAGS="${CFLAGS} -static"
	LDFLAGS="${LDFLAGS} -static"

	package/compile
}

package() {
	local x
	cd $pkgname-$pkgver
	#binaries
	install -dm755 "${pkgdir}/usr/bin"
	install -Dm755 nitro ${pkgdir}/usr/bin/nitro
	install -Dm755 nitroctl ${pkgdir}/usr/bin/nitroctl
	install -Dm755 ${srcdir}/shutdown ${pkgdir}/usr/bin/shutdown
	for x in halt poweroff reboot; do ln -s shutdown ${pkgdir}/usr/bin/$x;done
	install -Dm755 ${srcdir}/nsm ${pkgdir}/usr/bin/nsm
	install -Dm755 ${srcdir}/admin/runit-2.2.0/command/chpst ${pkgdir}/usr/bin/chpst

	# man pages
	install -dm755 "${pkgdir}/usr/share/man/man1"
	install -dm755 "${pkgdir}/usr/share/man/man8"
	install -Dm644 ${srcdir}/${pkgname}-${pkgver}/nitroctl.1 "${pkgdir}/usr/share/man/man8"
	install -Dm644 ${srcdir}/${pkgname}-${pkgver}/nitro.8 "${pkgdir}/usr/share/man/man1"

	cd $srcdir/runit-rc
	make DESTDIR="${pkgdir}" install-rc

	# iputils-specific configuration
	mkdir -p "$pkgdir/usr/lib/sysctl.d"
	echo "net.ipv4.ping_group_range = 0 2147483647" > "$pkgdir/usr/lib/sysctl.d/55-iputils.conf"

	install -dm755  "${pkgdir}/etc"
	cp -pr $srcdir/etc-nitro $pkgdir/etc/nitro
}
