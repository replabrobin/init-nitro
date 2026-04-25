pkgname=init-nitro
pkgver=0.8.2.1
#reviisions nitro, init-nitro-rc init-intro-base-svcs
_nrev="c937fa4fd202074dd250086ebd92de6884968b84"
_rcrev="e1e3b72ee25832f34f9ffde3ca96a81fea69ec03"
_bsrev="612105daef1aef63e4f8c4e7abb3cc84be4f9880"
_rver=2.2.0
_rpver=20250506
pkgrel=2
pkgdesc='simple init'
arch=('x86_64' 'aarch64')
url='https://github.com/leahneukirchen/nitro'
license=('GPL')
provides=('nitro' 'eudev-nitro')
depends=('glibc' 'bash' 'nitro-nsm' 'eudev-nitro' 'artix-cgroups')
conflicts=('svc-manager' 'eudev-nitro')
replaces=('eudev-nitro')
makedepends=()
optdepends=()
install=nitro.install
groups=()
backup=()
options=('!debug')
source=(
		"nitro-$pkgver-${_nrev:0:10}.tar.gz::https://github.com/leahneukirchen/nitro/archive/${_nrev}.tar.gz"
		"init-nitro-rc-${_rcrev:0:10}.tar.gz::https://github.com/replabrobin/init-nitro-rc/archive/${_rcrev}.tar.gz"
		"init-nitro-base-svcs-${_bsrev:0:10}.tar.gz::https://github.com/replabrobin/init-nitro-base-svcs/archive/${_bsrev}.tar.gz"
		"http://smarden.org/runit/runit-${_rver}.tar.gz"
		"runit-patches-${_rpver}.tar.xz::https://github.com/clan/runit/releases/download/runit-${_rver}-r2/runit-${_rver}-patches-${_rpver}.tar.xz"
		"000-services.patch"
		"shutdown"
		"nitro-install.hook"
		"nitro-remove.hook"
		"30-binfmt.hook"
		"30-sysctl.hook"
		"nitro-hook"
		"nitro-tmpfiles.conf"
		"rm-old-proc-1-exe.start"
		"version.txt.in"
		)
sha256sums=('47514091748c65b5cf48b2e5f8e53975f7d0882998ace5b00023ed10df08ae8f'
            'f837a384b64e538c6489ff708c7a1c39a9ceca6e47b2130eecb6fa6470a0c096'
            'd0d72c56437a8d8b1c215ff3aefc67f9517aa3f57d78758c390c0bbafaca052e'
            '95ef4d2868b978c7179fe47901e5c578e11cf273d292bd6208bd3a7ccb029290'
            'bbd115a9612c5a8df932cd43c406393538389b248ad44f1d9903bc0e2850e173'
            '1540738cb125828524b277bdfcd5804abbfa247b23d68aa1e28303dd1e39bf26'
            '08e048595bfac34ef656350c320a93de023e4b0e030a29bddaef3239d9d83d17'
            'f97f13a3ced1ed1b8ed35777957c284cd01be9f34a00f58950aafa3fe1c904c5'
            'd00df027e6805ac94fd0a4eed31cd2a907897bcf1cf053c698412d4c0dc27cae'
            'c74fa59269b3a7eeca785858de8e267c951169d3384bf543d8850f9594e049f1'
            '070114ee451b1842c6e7663efb16c8a8da09eda95aa38de00108dd123e435f53'
            '3aeee4a49cbe175d830375c2ecaaa372f18d1753e21a8b820092559baf1e618c'
            '071618745ff8a8a4473747dab599375beec88555706cae8d75f733b8eb3d8f28'
            'e7341f0e69dd20924126419131f84ed67a5f0e697006b187d0edde900b8a90b1'
            '2940cec3306bc3907bcc48ccbdc4cf750a7324037f06b8e4d7d0d928ca5b606e')
validpgpkeys=()


prepare() {
	local x
	arch='$arch' \
	_inrev="$(git rev-parse HEAD)" \
	_nrev="$_nrev" \
	_bsrev="$_bsrev" \
	_rcrev="$_rcrev" \
	version="${pkgver}-${pkgrel}-${_nrev:0:10}" \
		envsubst < version.txt.in > version.txt
	ln -sf nitro-${_nrev} nitro
	ln -sf init-nitro-rc{-${_rcrev},}
	cd "$srcdir/nitro"
	patch -Np1 -i "$srcdir"/000-services.patch

	cd "$srcdir/admin/runit-${_rver}/src"
	for x in "${srcdir}"/patches/*; do
		if [ "$VERBOSE" = 1 ]; then
			patch -Np2 -i "$x" 
		else
			patch -Np2 -i "$x" | (grep -e "chpst" -e"utmpset" || true)
		fi
	done
	#make a copy so we can mess with base-svcs
	rm -rf "$srcdir/base-svcs"
	cp -pr "$srcdir/init-nitro-base-svcs-${_bsrev}/" "$srcdir/base-svcs"
	rm -rf "$srcdir/base-svcs/"{LICENSE,.git}
}

build() {
	local icflags="${CFLAGS}"
	cd nitro
	[ -n "$DEBUG" ] && CFLAGS="${icflags} -DDEBUG=${DEBUG}"
	make
	CFLAGS="${icflags}"
	cd "${srcdir}/init-nitro-rc"
	make RCRUNDIR=/run/nitro/sv.d
	#from runit pkg
	#cc ${CFLAGS} ${srcdir}/halt.c -o ${srcdir}/halt ${LDFLAGS}

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
	cd nitro
	#binaries
	install -dm755 "${pkgdir}"/usr/bin
	install -Dm755 nitro "${pkgdir}"/usr/bin/nitro
	install -Dm755 nitroctl "${pkgdir}"/usr/bin/nitroctl
	install -Dm755 "${srcdir}"/shutdown "${pkgdir}"/usr/bin/shutdown
	install -Dm755 "${srcdir}"/nitro-hook "${pkgdir}"/usr/share/libalpm/scripts/nitro-hook
	install -Dm644 "${srcdir}"/nitro-install.hook "${pkgdir}"/usr/share/libalpm/hooks/nitro-install.hook
	install -Dm644 "${srcdir}"/nitro-remove.hook "${pkgdir}"/usr/share/libalpm/hooks/nitro-remove.hook
	install -Dm644 "${srcdir}"/30-binfmt.hook "${pkgdir}"/usr/share/libalpm/hooks/30-binfmt.hook
	install -Dm644 "${srcdir}"/30-sysctl.hook "${pkgdir}"/usr/share/libalpm/hooks/30-sysctl.hook
	for x in halt poweroff reboot; do ln -s nitroctl "${pkgdir}"/usr/bin/"$x";done
	install -Dm755 "${srcdir}"/admin/runit-"${_rver}"/command/chpst "${pkgdir}"/usr/bin/chpst
	install -Dm755 "${srcdir}"/admin/runit-"${_rver}"/command/utmpset "${pkgdir}"/usr/bin/utmpset

	# man pages
	install -dm755 "${pkgdir}"/usr/share/man/man1
	install -dm755 "${pkgdir}"/usr/share/man/man8
	install -Dm644 "${srcdir}"/nitro/nitroctl.1 "${pkgdir}"/usr/share/man/man1/nitroctl.1
	install -Dm644 "${srcdir}"/nitro/nitro.8 "${pkgdir}"/usr/share/man/man8/nitro.8
	install -Dm644 "${srcdir}"/admin/runit-"${_rver}"/man/chpst.8 "${pkgdir}"/usr/share/man/man8/chpst.8
	install -Dm644 "${srcdir}"/admin/runit-"${_rver}"/man/utmpset.8 "${pkgdir}"/usr/share/man/man8/utmpset.8

	cd $srcdir/init-nitro-rc
	make DESTDIR="${pkgdir}" install-rc
	install -Dm644 "${srcdir}"/init-nitro-rc/script/zzz.8 "${pkgdir}"/usr/share/man/man8/zzz.8
	install -Dm755 "${srcdir}"/init-nitro-rc/script/zzz "${pkgdir}"/usr/bin/zzz
	install -Dm644 "${srcdir}"/init-nitro-rc/src/pause.1 "${pkgdir}"/usr/share/man/man1/pause.1
	install -Dm755 "${srcdir}"/init-nitro-rc/src/pause "${pkgdir}"/usr/bin/pause
	install -dm755 "${pkgdir}"/usr/lib/tmpfiles.d
	install -Dm644 "${srcdir}"/nitro-tmpfiles.conf "${pkgdir}"/usr/lib/tmpfiles.d/nitro-tmpfiles.conf
	install -dm755 "${pkgdir}"/etc/local.d
	install -Dm755 "${srcdir}"/rm-old-proc-1-exe.start "${pkgdir}"/etc/local.d/rm-old-proc-1-exe.start


	# iputils-specific configuration
	mkdir -p "$pkgdir"/usr/lib/sysctl.d
	echo "net.ipv4.ping_group_range = 0 2147483647" > "$pkgdir"/usr/lib/sysctl.d/55-iputils.conf

	install -dm755  "${pkgdir}"/etc
	cp -pr "$srcdir"/base-svcs "$pkgdir"/etc/nitro
	install -Dm644 "${srcdir}"/version.txt "${pkgdir}"/etc/nitro/version.txt
}
