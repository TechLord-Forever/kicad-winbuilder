# Maintainer: Alexey Pavlov <alexpux@gmail.com>

_realname=kicad
pkgname="${MINGW_PACKAGE_PREFIX}-${_realname}-git"
pkgver=r7029.cc2b358
pkgrel=1
pkgdesc="Software for the creation of electronic schematic diagrams and printed circuit board artwork (mingw-w64)"
arch=('any')
url="http://www.kicad-pcb.org"
license=("GPL2+")
provides=("${MINGW_PACKAGE_PREFIX}-${_realname}")
conflicts=("${MINGW_PACKAGE_PREFIX}-${_realname}")
depends=("${MINGW_PACKAGE_PREFIX}-boost"
         "${MINGW_PACKAGE_PREFIX}-cairo"
         "${MINGW_PACKAGE_PREFIX}-curl"
         "${MINGW_PACKAGE_PREFIX}-glew"
         "${MINGW_PACKAGE_PREFIX}-openssl"
         "${MINGW_PACKAGE_PREFIX}-wxPython"
         "${MINGW_PACKAGE_PREFIX}-wxWidgets"
         "${MINGW_PACKAGE_PREFIX}-libxslt"
         "${MINGW_PACKAGE_PREFIX}-oce"
         "${MINGW_PACKAGE_PREFIX}-ngspice-git")

makedepends=("${MINGW_PACKAGE_PREFIX}-cmake"
             "${MINGW_PACKAGE_PREFIX}-doxygen"
             "${MINGW_PACKAGE_PREFIX}-gcc"
             "${MINGW_PACKAGE_PREFIX}-python2"
             "${MINGW_PACKAGE_PREFIX}-pkg-config"
             "${MINGW_PACKAGE_PREFIX}-swig"
             "${MINGW_PACKAGE_PREFIX}-glm"
             "git"
             "unzip")
source=("${_realname}"::"git+https://github.com/KiCad/kicad-source-mirror.git"
        "${_realname}-i18n"::"git+https://github.com/KiCad/kicad-i18n.git"
       )
md5sums=('SKIP'
         'SKIP'
        )
noextract=("${_realname}-footprints.zip")

pkgver() {
  cd "${srcdir}/${_realname}"
  printf "r%s.%s" "$(git rev-list --count --first-parent HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  if [ -d ${srcdir}/arch ]; then
    rm -rf ${srcdir}/archive
  fi
  # Hack to get UTF8 support for paths with mingw
  # https://lists.launchpad.net/kicad-developers/msg28560.html
  rm -rf ${srcdir}/${_realname}/include/wx
  mkdir ${srcdir}/${_realname}/include/wx
  #cp $(find /usr/x86_64-*mingw*/ -path "*wx\\/app.h") \
  cp $(find ${MINGW_PREFIX} -path "*wx\\/app.h") \
    ${srcdir}/${_realname}/include/wx
  cd ${srcdir}/${_realname}
  patch -p1 -i ${srcdir}/${_realname}/patches/wxwidgets-3.0.2_mingw_fix_unicode_entry.patch
}

build() {
  cd "${srcdir}"

  # Configure and build KiCad.
  [[ -d build-${MINGW_CHOST} ]] && rm -r build-${MINGW_CHOST}
  mkdir build-${MINGW_CHOST} && cd build-${MINGW_CHOST}
  
  # Get GCC version
  GCCVERSION=`gcc --version | grep ^gcc | sed 's/^.* //g'`

  # Add flag to silence deprecation warnings
  # Due to bug in gcc 5.1,5.2
  # https://gcc.gnu.org/bugzilla/show_bug.cgi?id=65974
  EXTRA_FLAGS=""
  if [ $GCCVERSION = "5.1.0" ] || [ $GCCVERSION = "5.2.0" ]; then
    EXTRA_FLAGS=" -DCMAKE_CXX_FLAGS=-Wno-deprecated-declarations"
  fi
  
  MSYS2_ARG_CONV_EXCL="-DCMAKE_INSTALL_PREFIX=" \
  ${MINGW_PREFIX}/bin/cmake.exe \
    -G"MSYS Makefiles" \
    -DCMAKE_BUILD_TYPE=RelWithDebInfo \
    -DCMAKE_PREFIX_PATH=${MINGW_PREFIX} \
    -DCMAKE_INSTALL_PREFIX=${MINGW_PREFIX} \
    -DDEFAULT_INSTALL_PATH=${MINGW_PREFIX} \
    -DOPENSSL_ROOT_DIR=${MINGW_PREFIX} \
    -DKICAD_SKIP_BOOST=ON \
    -DKICAD_SCRIPTING=ON \
    -DKICAD_SCRIPTING_MODULES=ON \
    -DKICAD_SCRIPTING_WXPYTHON=ON \
    -DKICAD_SCRIPTING_ACTION_MENU=ON \
    -DKICAD_USE_OCE=ON \
    -DKICAD_USE_SCH_IO_MANAGER=OFF \
    -DKICAD_SPICE=ON \
    -DPYTHON_EXECUTABLE=${MINGW_PREFIX}/bin/python2.exe \
    ${EXTRA_FLAGS} \
    ../${_realname}
  make

  cd "${srcdir}"

  # Configure the translation installation build.
  [[ -d build-i18n ]] && rm -r build-i18n
  mkdir build-i18n && cd build-i18n
  MSYS2_ARG_CONV_EXCL="-DCMAKE_INSTALL_PREFIX=" \
  ${MINGW_PREFIX}/bin/cmake.exe \
    -G "MSYS Makefiles" \
    -DCMAKE_INSTALL_PREFIX=${MINGW_PREFIX} \
    ../${_realname}-i18n

  cd "${srcdir}"

}

package() {
  # Install KiCad.
  cd "${srcdir}/build-${MINGW_CHOST}"
  make DESTDIR=${pkgdir} install

  # Install KiCad i18n.
  cd "${srcdir}/build-i18n"
  make DESTDIR=${pkgdir} install

}
