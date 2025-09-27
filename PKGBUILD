pkgname=init-nitro
pkgver=0.4.1
_nrev="150ca52ac0fe880deeaf6f899502820d4db82b6e"
_rcrev="e0c4e306e448d2ac7a8a314133995ee37bc48f92a"
_rver=2.2.0
_rpver=20250506
pkgrel=1
pkgdesc='simple init'
arch=('x86_64' 'aarch64')
url='https://github.com/leahneukirchen/nitro'
license=('GPL')
provides=('nitro')
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
		"00-init-nitro-remove.hook"
		"99-init-nitro-install.hook"
		"init-nitro-script"
		)
sha256sums=('557688775cbdcd9981526aaad79f29b900f425e534db096eb48daecda4c0483a'
            'SKIP'
            '95ef4d2868b978c7179fe47901e5c578e11cf273d292bd6208bd3a7ccb029290'
            'bbd115a9612c5a8df932cd43c406393538389b248ad44f1d9903bc0e2850e173'
            '191f7d0ad00183ab3a8820ca9bf4295de8af6d9bfa06571653e8fd3d8280e63a'
            '3be45d4a6e42ca33bec1ed86dcbfabd70219ab670f3a1dab8bde6cd3ddfdb1d2'
            'e453e6ffe63e128d1ae4708f1b380f711872e84d42c91492cb8a32a991d6fdc8'
            '42e5c1fbbc9aa7075a48d29e03d3f4f143d9ab7a517a540c575e6994473c645d'
            '07aecac5688b90e9ba4c0b169175fc8d359a393b7f011ae39cac570242bdb906'
            'bad320acb9dd3aca65a0c835bf4693f8db1e677557b99744a0869924dd4fcbc7'
            'c5d1acec2129a16bdc367b8b7aae9645174d940276fb9519832c7098e488c528'
            '50706e557b8f5dcd451f70b7f86c71a2c7cb78efcc7d9726c15f36d6a773f380'
            '979b592f12348e49f7543ad539780af5b6fa7b8d3c3669ca6d683894128be26f')
validpgpkeys=()

prepare() {
	local x
	ln -sf nitro-${_nrev} $pkgname-$pkgver
	cd $pkgname-$pkgver
	patch -Np1 -i "$srcdir"/000-services.patch
	cd "$srcdir/runit-rc"
	#patch -Np1 -i "$srcdir"/001-runit-rc.patch

	cd "$srcdir/admin/runit-${_rver}/src"
	for x in "${srcdir}"/patches/*; do
		if [ "$VERBOSE" = 1 ]; then
			patch -Np2 -i "$x" 
		else
			patch -Np2 -i "$x" | (grep -e "chpst" -e"utmpset" || true)
		fi
	done
}

build() {
	local icflags="${CFLAGS}"
	cd $pkgname-$pkgver
	[ -n "$DEBUG" ] && CFLAGS="${icflags} -DDEBUG=${DEBUG}"
	make
	CFLAGS="${icflags}"
	cd "${srcdir}/runit-rc"
	make
	#from runit pkg
	#cc ${CFLAGS} halt.c -o halt ${LDFLAGS}

	#cd ${_pkgname}
	#make SERVICEDIR="${_servicedir}"

	cd "${srcdir}/admin/runit-${_rver}"

	CFLAGS="${CFLAGS} -static"
	LDFLAGS="${LDFLAGS} -static"

	if [ "$VERBOSE" = 1 ]; then
		package/compile
	else
		package/compile |& (grep -e "chpst" -e "utmpset" || true)
	fi
}

package() {
	local x
	cd $pkgname-$pkgver
	#binaries
	install -dm755 "${pkgdir}/usr/bin"
	install -Dm755 nitro ${pkgdir}/usr/bin/nitro
	install -Dm755 nitroctl ${pkgdir}/usr/bin/nitroctl
	install -Dm755 ${srcdir}/shutdown ${pkgdir}/usr/bin/shutdown
	install -Dm755 ${srcdir}/init-nitro-script "${pkgdir}/usr/share/libalpm/scripts/init-nitro-script"
	install -Dm644 ${srcdir}/00-init-nitro-remove.hook "${pkgdir}/usr/share/libalpm/hooks/00-init-nitro-remove.hook"
	install -Dm644 ${srcdir}/99-init-nitro-install.hook "${pkgdir}/usr/share/libalpm/hooks/99-init-nitro-install.hook"
	for x in halt poweroff reboot; do ln -s shutdown ${pkgdir}/usr/bin/$x;done
	install -Dm755 ${srcdir}/nsm ${pkgdir}/usr/bin/nsm
	install -Dm755 ${srcdir}/admin/runit-${_rver}/command/chpst ${pkgdir}/usr/bin/chpst
	install -Dm755 ${srcdir}/admin/runit-${_rver}/command/utmpset ${pkgdir}/usr/bin/utmpset

	# man pages
	install -dm755 "${pkgdir}/usr/share/man/man1"
	install -dm755 "${pkgdir}/usr/share/man/man8"
	install -Dm644 ${srcdir}/${pkgname}-${pkgver}/nitroctl.1 "${pkgdir}/usr/share/man/man1/nitroctl.1"
	install -Dm644 ${srcdir}/${pkgname}-${pkgver}/nitro.8 "${pkgdir}/usr/share/man/man8/nitro.8"
	install -Dm644 ${srcdir}/admin/runit-${_rver}/man/chpst.8 "${pkgdir}/usr/share/man/man8/chpst.8"
	install -Dm644 ${srcdir}/admin/runit-${_rver}/man/utmpset.8 "${pkgdir}/usr/share/man/man8/utmpset.8"

	cd $srcdir/runit-rc
	make DESTDIR="${pkgdir}" install-rc

	# iputils-specific configuration
	mkdir -p "$pkgdir/usr/lib/sysctl.d"
	echo "net.ipv4.ping_group_range = 0 2147483647" > "$pkgdir/usr/lib/sysctl.d/55-iputils.conf"

	install -dm755  "${pkgdir}/etc"
	cp -pr $srcdir/etc-nitro $pkgdir/etc/nitro
}
