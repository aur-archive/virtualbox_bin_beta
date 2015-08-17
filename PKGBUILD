# Contributor: Det <nimetonmaili at gmail a-dot com>
# Contributor: The Ringmaster <theringmaster AT archlinux.us>
# Contributor: Christian Berendt <berendt@b1-systems.de>
# Contributor: Balwinder S "bsd" Dheeman <bdheeman AT gmail.com>
# Contributor: thotypous <matiasÎ˜archlinux-brÂ·org>
# Based on virtualbox_bin and virtualbox-ext-oracle

pkgbase=virtualbox_bin_beta
pkgname=${pkgbase}
true && pkgname=('virtualbox_bin_beta' 'virtualbox-ext-oracle-beta')
pkgver=4.1.4
_build=74291
pkgrel=1
arch=('i686' 'x86_64')
url='http://virtualbox.org'
license=('GPL2' 'custom:PUEL')
options=('!strip')
_arch='x86'
[ "${CARCH}" = 'x86_64' ] && _arch='amd64'
source=(http://download.virtualbox.org/virtualbox/${pkgver}/VirtualBox-${pkgver}-${_build}-Linux_${_arch}.run
        http://download.virtualbox.org/virtualbox/$pkgver/Oracle_VM_VirtualBox_Extension_Pack-$pkgver-$_build.vbox-extpack
        '10-vboxdrv.rules'
        'vboxdrv.rc'
        'vboxdrv.conf'
        'vboxweb.rc'
        'vboxweb.conf'
        'PUEL')
md5sums=(`wget ${source/V*}MD5SUMS -qO - | grep ${_arch}.run | cut -d " " -f1`
         `wget ${source[1]/O*}MD5SUMS -qO - | grep VM | cut -d " " -f1`
         'fe60f9510502bea67383d9198ae8c13c'
         'e7a94c97b1b1e1e843c1bb85181d2de8'
         '951bf4a5524e919fc4aaee6f46041e3d'
         'c159d683ba1947290fc2ad2c64194150'
         '3ac185709bfe688bb753c46e170d0546'
         '08b28b82d0ebd6962025100d4b5414a1')

package_virtualbox_bin_beta() {
  pkgdesc="A powerful x86 virtualizer (Personal Use Edition, bleeding edge binaries)"
  depends=('libidl2' 'libxcursor' 'libxinerama' 'libxslt' 'curl' 'linux-headers' 'python2')
  optdepends=('virtualbox-ext-oracle-beta: for Oracle extensions'
              'dkms: for handling kernel modules with dkms'
              'qt: for GUI support'
              'sdl: for VBoxSDL and GUI support'
              'mesa: for OpenGL support'
              'libgl: for shared OpenGL support'
              'libxt: for shared clipboard support'
              'alsa-lib: for ALSA support'
              'pulseaudio: for PulseAudio support')
  provides=("virtualbox=${pkgver}")
  conflicts=('virtualbox' 'virtualbox-modules')
  backup=('etc/vbox/vbox.cfg' 'etc/conf.d/vboxdrv' 'etc/conf.d/vboxweb')
  install=${pkgbase}.install
  _installdir=/opt/VirtualBox

  # Unpack the run package via sh(1)
  echo yes | sh VirtualBox-${pkgver}-${_build}-Linux_${_arch}.run --target . --nox11 --noexec &> /dev/null

  # Unpack bundled files
  install -d "${pkgdir}/${_installdir}"
  cd "${pkgdir}/${_installdir}"
  tar -xjf "${srcdir}/VirtualBox.tar.bz2"

  # Hardened build: Mark binaries suid root, create symlinks for working around
  #                 unsupported $ORIGIN/.. in VBoxC.so and make sure the
  #                 directory is only writable by the user (paranoid).
  chmod 4511 VirtualBox VBox{SDL,Headless,NetDHCP,NetAdpCtl}
  for _lib in VBox{VMM,REM,RT,DDU,XPCOM}.so; do
    ln -sf ${_installdir}/${_lib} components/${_lib}
  done
  chmod go-w .

  # Patch "vboxshell.py" to use Python 2.x instead of Python 3
  sed -i 's#/usr/bin/python#\02#' "${pkgdir}/${_installdir}/vboxshell.py"

  # Update Arch initscripts way of life in VBox.sh
  sed -i -e 's,sudo /etc/init.d/vboxdrv setup,/etc/rc.d/vboxdrv setup,g' "${pkgdir}/${_installdir}/VBox.sh"
  sed -i -e 's,sudo /etc/init.d/vboxdrv restart,/etc/rc.d/vboxdrv restart,g' "${pkgdir}/${_installdir}/VBox.sh"

  # Install vboxdrv initscript
  install -D -m 0755 "${srcdir}/vboxdrv.rc" "${pkgdir}/etc/rc.d/vboxdrv"
  install -D -m 0644 "${srcdir}/vboxdrv.conf" "${pkgdir}/etc/conf.d/vboxdrv"

  # Install vboxweb initscript
  install -D -m 0755 "${srcdir}/vboxweb.rc" "${pkgdir}/etc/rc.d/vboxweb"
  install -D -m 0644 "${srcdir}/vboxweb.conf" "${pkgdir}/etc/conf.d/vboxweb"

  # Install udev rules
  install -D -m 0644 "${srcdir}/10-vboxdrv.rules" "${pkgdir}/lib/udev/rules.d/10-vboxdrv.rules"
  ln -s "${_installdir}/VBoxCreateUSBNode.sh" "${pkgdir}/lib/udev/VBoxCreateUSBNode.sh"

  # Install the SDK
  cd sdk/installer
  VBOX_INSTALL_PATH=${_installdir} python2 vboxapisetup.py install --root "${pkgdir}"
  rm -r -f build
  cd -

  # Symlink the launchers. Second link can fail if fs is not case sensitive.
  install -d -m 0755 "${pkgdir}/usr/bin"
  for _bin in VirtualBox VBox{Headless,Manage,SDL,SVC,Tunctl,NetAdpCtl} rdesktop-vrdp; do
    ln -s ${_installdir}/${_bin} "${pkgdir}/usr/bin/${_bin}"
    ln -s ${_installdir}/${_bin} "${pkgdir}/usr/bin/${_bin,,}" &>/dev/null || :
  done

  # Symlink the desktop icon and ".desktop" files
  install -d -m 0755 "${pkgdir}/usr/"{share/applications,share/pixmaps}
  ln -s ${_installdir}/VBox.png "${pkgdir}/usr/share/pixmaps/VBox.png"
  ln -s ${_installdir}/icons/128x128/virtualbox.png "${pkgdir}/usr/share/pixmaps/virtualbox.png"
  ln -s ${_installdir}/virtualbox.desktop "${pkgdir}/usr/share/applications/virtualbox.desktop"

  # Symlink mime info
  install -d -m 0755 "${pkgdir}/usr/share/mime/packages"
  ln -s ${_installdir}/virtualbox.xml "${pkgdir}/usr/share/mime/packages/virtualbox.xml"

  # Symlink doc
  install -d -m 0755 "${pkgdir}/usr/share/doc/${pkgname}"
  ln -s ${_installdir}/VirtualBox.chm "${pkgdir}/usr/share/doc/$pkgname/virtualbox.chm"

  # Symlink module sources
  install -d -m 0755 "${pkgdir}/usr/src"
  ln -s ${_installdir}/src/vboxhost "${pkgdir}/usr/src/vboxhost-${pkgver}"

  # Symlink icons
  cd "${pkgdir}/${_installdir}/icons"
  for _dir in *; do
    cd "${_dir}"
    install -d -m 0755 "${pkgdir}/usr/share/icons/hicolor/${_dir}/"{apps,mimetypes}
    for _icon in *; do
      if [[ "${_icon}" = 'virtualbox.png' ]]; then
          ln -s ${_installdir}/icons/${_dir}/${_icon} "${pkgdir}/usr/share/icons/hicolor/${_dir}/apps/${_icon}"
      else
          ln -s ${_installdir}/icons/${_dir}/${_icon} "${pkgdir}/usr/share/icons/hicolor/${_dir}/mimetypes/${_icon}"
      fi
    done
    cd - >/dev/null
  done

  # Write the configuration file
  install -d -m 0755 "${pkgdir}/etc/vbox"
  echo "# VirtualBox installation directory" > "${pkgdir}/etc/vbox/vbox.cfg"
  echo "INSTALL_DIR=${_installdir}" >> "${pkgdir}/etc/vbox/vbox.cfg"
  echo "# VirtualBox version" >> "${pkgdir}/etc/vbox/vbox.cfg"
  echo "INSTALL_VER='${pkgver}'" >> "${pkgdir}/etc/vbox/vbox.cfg"
  echo "INSTALL_REV='${_build}'" >> "${pkgdir}/etc/vbox/vbox.cfg"
  chmod 0644 "${pkgdir}/etc/vbox/vbox.cfg"

  # Create the directory below if it doesn't exist
  install -d -m 0755 "${pkgdir}/var/run/VirtualBox"
}

package_virtualbox-ext-oracle-beta() {
  pkgdesc="An extension pack for Virtualbox (bleeding edge)"
  arch=('any')
  depends=("$pkgbase=$pkgver")
  install=virtualbox-ext-oracle-beta.install

  install -D -m 644 Oracle_VM_VirtualBox_Extension_Pack-$pkgver-$_build.vbox-extpack "$pkgdir/usr/share/virtualbox/extensions/Oracle_VM_VirtualBox_Extension_Pack-$pkgver.vbox-extpack"
  install -D -m 644 PUEL "$pkgdir/usr/share/licenses/$pkgname/PUEL"
}

pkgdesc="A powerful x86 virtualizer (Personal Use Edition + Extension Pack, bleeding edge binaries)"
depends=('libidl2' 'libxcursor' 'libxinerama' 'libxslt' 'curl' 'kernel26-headers' 'python2')

# vim:set ts=2 sw=2 ft=sh et: