# Maintainer: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Xavion <Xavion (dot) 0 (at) Gmail (dot) com>
# Contributor: Calogero Lo Leggio <kalos@autistici.org>
# Contributor: Matias Hernandez <msdark@archlinux.cl>

pkgname=bacula-postgresql
_pkgname=bacula
pkgver=5.2.13
pkgrel=4
pkgdesc="An advanced backup tool with network and tape changer support (PostgreSQL backend)"
arch=("i686" "x86_64")
url="http://www.${_pkgname}.org"
license=("GPL")
depends=('openssl' 'lzo2' 'zlib' 'python2' 'postgresql-libs')
makedepends=("qt4" "gtk2")
optdepends=("qt4: for bat"
            "gtk2: for tray monitor"
            "python2: python support")
options=(!buildflags !libtool)
conflicts=("${_pkgname}-sqlite" "${_pkgname}-mysql" "${_pkgname}" "${_pkgname}-client")
backup=("etc/${_pkgname}/bconsole.conf"
		"etc/${_pkgname}/${_pkgname}-dir.conf"
		"etc/${_pkgname}/${_pkgname}-fd.conf"
		"etc/${_pkgname}/${_pkgname}-sd.conf")
install="${_pkgname}.install"
source=(http://downloads.sourceforge.net/project/bacula/bacula/${pkgver}/${_pkgname}-${pkgver}.tar.gz
        ${_pkgname}-sd.rc.d
        ${_pkgname}-fd.rc.d
        ${_pkgname}-dir.rc.d
		${_pkgname}.conf)
md5sums=('43417bae0c221afb1f30a581c9e0f2fe'
         '6311f10c58261c4ee6e26ae2de5580a3'
         '7c240ed89c6e42b379386d27e163d3bc'
         'bb31d783b3362961eac4746219e3e3ff'
         'd5357537e6535f1fee62fa27baa9fa42')



build() {
	cd ${srcdir}/${_pkgname}-${pkgver}

    sed -i "s/python-config/python2-config/g" configure

	# Build
	QMAKE=/usr/bin/qmake-qt4 ./configure --prefix=/usr --sbindir=/usr/bin \
		--enable-build-dird --enable-build-stored --enable-smartalloc \
		--enable-bat --enable-tray-monitor \
        --enable-batch-insert --enable-ipv6 \
		--with-postgresql --with-openssl --with-python --with-tcp-wrappers=no \
        --with-systemd \
		--with-fd-user=${_pkgname} --with-fd-group=${_pkgname} \
		--with-dir-user=${_pkgname} --with-dir-group=${_pkgname} \
		--with-sd-user=${_pkgname} --with-sd-group=${_pkgname} \
		--sysconfdir=/etc/${_pkgname} --with-scriptdir=/etc/${_pkgname}/scripts \
		--with-working-dir=/var/cache/${_pkgname}/working \
		--with-subsys-dir=/var/cache/${_pkgname}/working \
		--with-archivedir=/var/cache/${_pkgname}/archive \
        --with-pid-dir=/run/bacula/

	make
}

package() {
	cd ${srcdir}/${_pkgname}-${pkgver}

	make DESTDIR=${pkgdir} install

    sed -i "s/var-run.mount //g" platforms/systemd/bacula-{dir,fd,sd}.service
    sed -i "s/syslog.target //g" platforms/systemd/bacula-{dir,fd,sd}.service
	install -Dm644 platforms/systemd/bacula-dir.service ${pkgdir}/usr/lib/systemd/system/bacula-dir.service
	install -Dm644 platforms/systemd/bacula-fd.service ${pkgdir}/usr/lib/systemd/system/bacula-fd.service
	install -Dm644 platforms/systemd/bacula-sd.service ${pkgdir}/usr/lib/systemd/system/bacula-sd.service

	# Permissions
	chmod a+x ${pkgdir}/etc/${_pkgname}/scripts/{update_${_pkgname}_tables,delete_catalog_backup,update_postgresql_tables,make_catalog_backup,bconsole}

	# Daemons
	mkdir -p ${pkgdir}/etc/rc.d/
    install -Dm755 ${srcdir}/bacula-dir.rc.d ${pkgdir}/etc/rc.d/bacula-dir
    install -Dm755 ${srcdir}/bacula-fd.rc.d ${pkgdir}/etc/rc.d/bacula-fd
    install -Dm755 ${srcdir}/bacula-sd.rc.d ${pkgdir}/etc/rc.d/bacula-sd

	# tmp dir for pid file
	mkdir -p ${pkgdir}/usr/lib/tmpfiles.d
    install -Dm644 ${srcdir}/${_pkgname}.conf ${pkgdir}/usr/lib/tmpfiles.d/${_pkgname}.conf

	# Logs
	install -D -m644 ${srcdir}/${_pkgname}-${pkgver}/scripts/logrotate ${pkgdir}/etc/logrotate.d/${_pkgname}
	sed -i "s|/var/cache/${_pkgname}/working/log|/var/log/${_pkgname}.log|g" ${pkgdir}/etc/{${_pkgname}/${_pkgname}-dir.conf,logrotate.d/${_pkgname}}

    # Fix permissions
    chmod -R 755 ${pkgdir}/usr/bin
}

