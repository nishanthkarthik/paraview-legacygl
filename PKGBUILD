# Maintainer: Bruno Pagani <archange@archlinux.org>
# Contributor: Oliver Goethel <deezy>
# Contributor: eolianoe eolianoe <eolianoe [at] gmail [DoT] com>
# Contributor: George Eleftheriou <eleftg>
# Contributor: Mathias Anselmann <mathias.anselmann@gmail.com>
# Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Michele Mocciola <mickele>
# Contributor: Simon Zilliken <simon____AT____zilliken____DOT____name>
# Contributor: chuckdaniels

_pkg=paraview
_mpi=openmpi
pkgname=${_pkg}-legacygl
#-${_mpi}
pkgver=5.4.1
pkgrel=1
pkgdesc="Parallel Visualization application using VTK (${_mpi} version)"
arch=('x86_64')
url="https://www.paraview.org"
license=('custom')
depends=('qt4' 'ospray' 'ffmpeg' 'openmpi'
         'cgns' 'python-pygments' 'protobuf' 'pugixml'
         'python-matplotlib' 'python-numpy' 'python-mpi4py'
         'python-six' 'python-constantly' 'python-twisted'
         'python-autobahn' 'python-zope-interface' 'python-incremental'
         'boost-libs' 'glew' 'expat' 'freetype2'
         'libjpeg' 'jsoncpp' 'libxml2' 'libtheora' 'libpng'
         'libtiff' 'zlib' 'hdf5-openmpi' 'lz4')
#        netcdf-cxx gl2ps libharu
#        python-txaio python-hyperlink
#        proj apparently not used in this VTK configuration
makedepends=('cmake' 'boost' 'mesa' 'gcc-fortran' 'ninja' 'qt5-tools' 'qt5-xmlpatterns')
source=("${url}/files/v${pkgver:0:3}/ParaView-v${pkgver}.tar.gz"
        'visit_fix_gcc7.patch'
        'jsoncpp-1.8.4.patch')
sha256sums=('390d0f5dc66bf432e202a39b1f34193af4bf8aad2355338fa5e2778ea07a80e4'
            'd1daa5da6ec25c5a6bfcabb3cf0bf02ab97ec87a332886a2f42695072fe8568d'
            'ed9a99d5d0fb54f7506e819f6d54bd1b6cd7dd0e91647d9d06591ae300f9ef05')

prepare() {
    mkdir -p build
    cd ParaView-v${pkgver}
    patch Utilities/VisItBridge/databases/readers/Vs/VsStaggeredField.C "${srcdir}"/visit_fix_gcc7.patch
    patch -p1 -i "${srcdir}"/jsoncpp-1.8.4.patch
}

build() {
    cd build

    # Flags to enable system libs in VTK building, as in VTK package
    # NETCDF NETCDFCPP status?
    # LIBPROJ4 UNUSED?
    # GL2PS fails.
    # libharu blocked by https://github.com/libharu/libharu/pull/157
    # TXAIO HYPERLINK in a future VTK version
    # LIBPROJ4 apparently not used in this VTK configuration
    local VTK_USE_SYSTEM_LIB=""
    for lib in EXPAT FREETYPE JPEG PNG TIFF ZLIB LIBXML2 OGGTHEORA TWISTED ZOPE SIX AUTOBAHN MPI4PY JSONCPP GLEW HDF5 CONSTANTLY INCREMENTAL LZ4
    do
        VTK_USE_SYSTEM_LIB+="-DVTK_USE_SYSTEM_${lib}:BOOL=ON "
    done
    # Specific system libs for ParaView version
    for lib in CGNS PUGIXML PROTOBUF PYGMENTS
    do
        VTK_USE_SYSTEM_LIB+="-DVTK_USE_SYSTEM_${lib}:BOOL=ON "
    done

    cmake ../ParaView-v${pkgver} \
        -DBUILD_DOCUMENTATION=OFF \
        -DBUILD_EXAMPLES=ON \
        -DBUILD_SHARED_LIBS=ON \
        -DBUILD_TESTING=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_COMPILER=mpicc \
        -DCMAKE_CXX_COMPILER=mpicxx \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DOSPRAY_INSTALL_DIR=/usr \
        -DPARAVIEW_ENABLE_FFMPEG=ON \
        -DPARAVIEW_ENABLE_MATPLOTLIB=ON \
        -DPARAVIEW_ENABLE_PYTHON=ON \
        -DPARAVIEW_INSTALL_DEVELOPMENT_FILES=ON \
        -DPARAVIEW_QT_VERSION=4 \
        -DPARAVIEW_USE_MPI=ON \
        -DPARAVIEW_USE_VISITBRIDGE=ON \
        -DPARAVIEW_USE_OSPRAY=ON \
        -DVISIT_BUILD_READER_CGNS=ON \
        -DVTK_PYTHON_FULL_THREADSAFE=ON \
        -DVTK_PYTHON_VERSION=3 \
        -DVTK_QT_VERSION=4 \
        -DVTK_RENDERING_BACKEND=OpenGL \
        -DVTK_SMP_IMPLEMENTATION_TYPE=OpenMP \
        ${VTK_USE_SYSTEM_LIB} \
        -GNinja

    ninja ${MAKEFLAGS}
}

package() {
    cd build

    DESTDIR="${pkgdir}" ninja install

    # Install license
    install -Dm644 "${srcdir}"/ParaView-v${pkgver}/License_v1.2.txt "${pkgdir}"/usr/share/licenses/paraview/LICENSE

    # Remove IceT man pages to avoid conflicts
    rm -- "${pkgdir}"/usr/share/man/man3/icet*.3
}
