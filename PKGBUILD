# $Id$
# Maintainer: Zarren Spry <zarrenspry at gmail dot com>
# Contributor: jcstryker <public at jasonstryker dot com>
# Contributor: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>
# Contributor: James Rayner <iphitus@gmail.com>
# Contributor: Alex Luberg <alex@luberg.me>

pkgbase=nvidia-vulkan
pkgname=('nvidia-vulkan' 'nvidia-vulkan-dkms' 'nvidia-vulkan-utils' 'opencl-nvidia-vulkan' 'lib32-nvidia-vulkan-utils' 'lib32-opencl-nvidia-vulkan')
pkgver="470.42.01"
pkgrel=1
pkgdesc='NVIDIA drivers for linux (vulkan)'
arch=('x86_64')
url="https://us.download.nvidia.com/"
makedepends=('libglvnd' 'linux' 'linux-headers')
license=('custom')
options=('!strip')
_pkg="NVIDIA-Linux-x86_64-${pkgver}"
source=("https://us.download.nvidia.com/XFree86/Linux-${CARCH}/${pkgver}/${_pkg}.run"
        'nvidia-drm-outputclass.conf'
        'nvidia-vulkan-utils.sysusers'
        '110-nvidia-change-dkms-conf.patch'
        '120-nvidia-linux-rt-gift.patch'
        '130-fix-regex-patterns-for-modulesDOTsymvers.patch')
sha512sums=('f933e249b9e0b043283ba164d56bd235240a34098e4ab9d1070809e17f8c2cd24d7319217c02c2417414f4c811216181d7ab8842755cd0d25b00ee51de70b2cc'
            'de7116c09f282a27920a1382df84aa86f559e537664bb30689605177ce37dc5067748acf9afd66a3269a6e323461356592fdfc624c86523bf105ff8fe47d3770'
            '4b3ad73f5076ba90fe0b3a2e712ac9cde76f469cd8070280f960c3ce7dc502d1927f525ae18d008075c8f08ea432f7be0a6c3a7a6b49c361126dcf42f97ec499'
            'e8afb7adbfe816372dff8a955e84f5b01adf32fb4a00abe72b7ee4e6ebd9f5a3e8bed4cbe0311da936054eeeaec93c12e16cdaab7f8c707bf875d3194ded6739'
            '5f254b86c1ecc0873d31e6c48f9ba7995f1268217e8c059f6b8fb254a43cb052358eb9195f83388486992774c0400d861b2aa33a48b78ae6eda1c30e497f000d'
            '007819cccaa919b5a44e7eb6201012d2d1cf3e5a26aacec505d5560d0973b2b63f9f83c7ded3a9fde611e8064d2cc3af5895fc5d38c0f4d305e00b4a2ee06e46')

create_links() {
    # create soname links
    find "$pkgdir" -type f -name '*.so*' ! -path '*xorg/*' -print0 | while read -d $'\0' _lib; do
        _soname=$(dirname "${_lib}")/$(readelf -d "${_lib}" | grep -Po 'SONAME.*: \[\K[^]]*' || true)
        _base=$(echo ${_soname} | sed -r 's/(.*).so.*/\1.so/')
        [[ -e "${_soname}" ]] || ln -s $(basename "${_lib}") "${_soname}"
        [[ -e "${_base}" ]] || ln -s $(basename "${_soname}") "${_base}"
    done
}

prepare() {
    sh "${_pkg}.run" --extract-only
    cd "${_pkg}"
    bsdtar -xf nvidia-persistenced-init.tar.bz2

    patch -Np1 -i "../110-nvidia-change-dkms-conf.patch"
    patch -Np1 -i "../120-nvidia-linux-rt-gift.patch"
    patch -Np1 -i "../130-fix-regex-patterns-for-modulesDOTsymvers.patch"

    cp -a kernel kernel-dkms

}

build() {
    cd "${_pkg}"/kernel
    make SYSSRC=/usr/src/linux module
}

package_nvidia-vulkan() {
    pkgdesc="NVIDIA drivers for linux (vulkan developer branch)"
    depends=('linux' "nvidia-vulkan-utils=${pkgver}" 'libglvnd')
    provides=("nvidia=$pkgver")
    conflicts+=('nvidia')
    
    _extradir="/usr/lib/modules/$(</usr/src/linux/version)/extramodules"
    install -Dt "${pkgdir}${_extradir}" -m644 \
      "${srcdir}/${_pkg}/kernel"/nvidia{,-modeset,-drm,-uvm}.ko

    find "${pkgdir}" -name '*.ko' -exec gzip -n {} +

    echo "blacklist nouveau" |
        install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkg}/LICENSE"
}

package_nvidia-vulkan-dkms() {
    pkgdesc="NVIDIA driver sources for linux (vulkan developer branch)"
    depends=('dkms' "nvidia-vulkan-utils=${pkgver}" 'libglvnd')
    provides=("nvidia=$pkgver")
    conflicts+=('nvidia')

    cd ${_pkg}

    install -dm 755 "${pkgdir}"/usr/src
    cp -dr --no-preserve='ownership' kernel-dkms "${pkgdir}/usr/src/nvidia-${pkgver}"

    echo "blacklist nouveau" |
        install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkg}/LICENSE"
}

package_opencl-nvidia-vulkan() {
    pkgdesc="OpenCL implemention for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=('opencl-driver' "opencl-nvidia=${pkgver}")
    conflicts=('opencl-nvidia')
    cd "${_pkg}"

    # OpenCL
    install -D -m644 nvidia.icd "${pkgdir}/etc/OpenCL/vendors/nvidia.icd"
    install -D -m755 "libnvidia-compiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-compiler.so.${pkgver}"
    install -D -m755 "libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opencl.so.${pkgver}"

    create_links

    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s nvidia "${pkgdir}/usr/share/licenses/opencl-nvidia-vulkan"
}

package_nvidia-vulkan-utils() {
    pkgdesc="NVIDIA drivers utilities"
    depends=('xorg-server' 'libglvnd' 'egl-wayland')
    optdepends=('gtk2: nvidia-settings (GTK+ v2)'
                'gtk3: nvidia-settings (GTK+ v3)'
                'xorg-server-devel: nvidia-xconfig'
                'opencl-nvidia-vulkan: OpenCL support')
    conflicts=('nvidia-utils' 'nvidia-libgl' 'nvidia-settings')
    provides=("nvidia-utils=${pkgver}" 'vulkan-driver' 'opengl-driver' 'nvidia-libgl' 'nvidia-settings')
    replaces=('nvidia-libgl')
    install="nvidia-vulkan-utils.install"

    cd "${_pkg}"

    # X driver
    install -D -m755 nvidia_drv.so "${pkgdir}/usr/lib/xorg/modules/drivers/nvidia_drv.so"

    # GLX extension module for X
    install -D -m755 "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.${pkgver}"

    # Ensure that X finds glx
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.1"
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so"

    install -D -m755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -D -m755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libEGL_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv1_CM_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv2_nvidia.so.${pkgver}"
    install -D -m644 "10_nvidia.json" "${pkgdir}/usr/share/glvnd/egl_vendor.d/10_nvidia.json"

    # OpenGL core library
    install -D -m755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glcore.so.${pkgver}"
    install -D -m755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-eglcore.so.${pkgver}"
    install -D -m755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glsi.so.${pkgver}"

    # misc
    install -D -m755 "libnvidia-ifr.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ifr.so.${pkgver}"
    install -D -m755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-fbc.so.${pkgver}"
    install -D -m755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-encode.so.${pkgver}"
    install -D -m755 "libnvidia-cfg.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-cfg.so.${pkgver}"
    install -D -m755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ml.so.${pkgver}"
    install -D -m755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glvkspirv.so.${pkgver}"

    # Vulkan ICD
    install -D -m644 "nvidia_icd.json" "${pkgdir}/usr/share/vulkan/icd.d/nvidia_icd.json"
    install -D -m644 "nvidia_layers.json" "${pkgdir}/usr/share/vulkan/implicit_layer.d/nvidia_layers.json"

    # VDPAU
    install -D -m755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -D -m755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -D -m755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib/libcuda.so.${pkgver}"
    install -D -m755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib/libnvcuvid.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -D -m755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ptxjitcompiler.so.${pkgver}"

    # raytracing
    install -D -m755 "libnvoptix.so.${pkgver}" "${pkgdir}/usr/lib/libnvoptix.so.${pkgver}"
    install -D -m755 "libnvidia-rtcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-rtcore.so.${pkgver}"
    install -D -m755 "libnvidia-cbl.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-cbl.so.${pkgver}"

    # Optical Flow
    install -D -m755 "libnvidia-opticalflow.so.${pkgver}" -t "${pkgdir}/usr/lib"

    # DEBUG
    install -D -m755 nvidia-debugdump "${pkgdir}/usr/bin/nvidia-debugdump"

    # nvidia-xconfig
    install -D -m755 nvidia-xconfig "${pkgdir}/usr/bin/nvidia-xconfig"
    install -D -m644 nvidia-xconfig.1.gz "${pkgdir}/usr/share/man/man1/nvidia-xconfig.1.gz"

    # nvidia-bug-report
    install -D -m755 nvidia-bug-report.sh "${pkgdir}/usr/bin/nvidia-bug-report.sh"

    # GTK+ for nvidia-settings
    install -Dm755 libnvidia-gtk2.so.$pkgver "$pkgdir"/usr/lib/libnvidia-gtk2.so.$pkgver
    install -Dm755 libnvidia-gtk3.so.$pkgver "$pkgdir"/usr/lib/libnvidia-gtk3.so.$pkgver

     # nvidia-settings
    install -Dm755 nvidia-settings "$pkgdir"/usr/bin/nvidia-settings
    install -Dm644 nvidia-settings.1.gz "$pkgdir"/usr/share/man/man1/nvidia-settings.1.gz
    install -Dm644 nvidia-settings.png "$pkgdir"/usr/share/pixmaps/nvidia-settings.png
    install -Dm644 nvidia-settings.desktop "$pkgdir"/usr/share/applications/nvidia-settings.desktop
    sed -e 's:__UTILS_PATH__:/usr/bin:' \
        -e 's:__PIXMAP_PATH__:/usr/share/pixmaps:' \
        -i "$pkgdir"/usr/share/applications/nvidia-settings.desktop

    # nvidia-smi
    install -D -m755 nvidia-smi "${pkgdir}/usr/bin/nvidia-smi"
    install -D -m644 nvidia-smi.1.gz "${pkgdir}/usr/share/man/man1/nvidia-smi.1.gz"

    # nvidia-cuda-mps
    install -D -m755 nvidia-cuda-mps-server "${pkgdir}/usr/bin/nvidia-cuda-mps-server"
    install -D -m755 nvidia-cuda-mps-control "${pkgdir}/usr/bin/nvidia-cuda-mps-control"
    install -D -m644 nvidia-cuda-mps-control.1.gz "${pkgdir}/usr/share/man/man1/nvidia-cuda-mps-control.1.gz"

    # nvidia-modprobe
    # This should be removed if nvidia fixed their uvm module!
    install -D -m4755 nvidia-modprobe "${pkgdir}/usr/bin/nvidia-modprobe"
    install -D -m644 nvidia-modprobe.1.gz "${pkgdir}/usr/share/man/man1/nvidia-modprobe.1.gz"

    # nvidia-persistenced
    install -D -m755 nvidia-persistenced "${pkgdir}/usr/bin/nvidia-persistenced"
    install -D -m644 nvidia-persistenced.1.gz "${pkgdir}/usr/share/man/man1/nvidia-persistenced.1.gz"
    install -D -m644 nvidia-persistenced-init/systemd/nvidia-persistenced.service.template "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"
    sed -i 's/__USER__/nvidia-persistenced/' "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"

    # application profiles
    install -D -m644 nvidia-application-profiles-${pkgver}-rc "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-rc"
    install -D -m644 nvidia-application-profiles-${pkgver}-key-documentation "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-key-documentation"

    install -D -m644 LICENSE "${pkgdir}/usr/share/licenses/nvidia-vulkan-utils/LICENSE"
    install -D -m644 README.txt "${pkgdir}/usr/share/doc/nvidia/README"
    install -D -m644 NVIDIA_Changelog "${pkgdir}/usr/share/doc/nvidia/NVIDIA_Changelog"
    cp -r html "${pkgdir}/usr/share/doc/nvidia/"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-vulkan-utils"

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -D -m644 "${srcdir}/nvidia-drm-outputclass.conf" "${pkgdir}/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf"

    install -Dm644 "${srcdir}/nvidia-vulkan-utils.sysusers" "${pkgdir}/usr/lib/sysusers.d/$pkgname.conf"

    create_links
}

package_lib32-opencl-nvidia-vulkan() {
    pkgdesc="OpenCL implemention for NVIDIA (32-bit)"
    depends=('lib32-zlib' 'lib32-gcc-libs')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=('lib32-opencl-driver' "lib32-opencl-nvidia=${pkgver}")
    conflicts=('lib32-opencl-nvidia' 'lib32-opencl-nvidia-390xx')

    cd "${_pkg}"/32

    # OpenCL
    install -D -m755 "libnvidia-compiler.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-compiler.so.${pkgver}"
    install -D -m755 "libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-opencl.so.${pkgver}"

    create_links

    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s lib32-nvidia-utils "${pkgdir}/usr/share/licenses/lib32-opencl-nvidia-vulkan"
}

package_lib32-nvidia-vulkan-utils() {
    pkgdesc="NVIDIA drivers utilities (32-bit)"
    depends=('lib32-zlib' 'lib32-gcc-libs' 'lib32-libglvnd' 'nvidia-vulkan-utils')
    optdepends=('lib32-opencl-nvidia-vulkan')
    conflicts=('lib32-nvidia-utils' 'lib32-nvidia-libgl')
    provides=("lib32-nvidia-utils=${pkgver}" 'lib32-vulkan-driver' 'lib32-opengl-driver' 'lib32-nvidia-libgl')
    replaces=('lib32-nvidia-libgl')

    cd "${_pkg}"/32

    install -D -m755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -D -m755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libEGL_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLESv1_CM_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLESv2_nvidia.so.${pkgver}"

    # OpenGL core library
    install -D -m755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glcore.so.${pkgver}"
    install -D -m755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-eglcore.so.${pkgver}"
    install -D -m755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glsi.so.${pkgver}"

    # misc
    install -D -m755 "libnvidia-ifr.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-ifr.so.${pkgver}"
    install -D -m755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-fbc.so.${pkgver}"
    install -D -m755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-encode.so.${pkgver}"
    install -D -m755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-ml.so.${pkgver}"
    #install -D -m755 "libnvidia-ngx.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ngx.so.${pkgver}"
    install -D -m755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glvkspirv.so.${pkgver}"

    # Optical Flow
    install -D -m755 "libnvidia-opticalflow.so.${pkgver}" -t "${pkgdir}/usr/lib32"

    # VDPAU
    install -D -m755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -D -m755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -D -m755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib32/libcuda.so.${pkgver}"
    install -D -m755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib32/libnvcuvid.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -D -m755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-ptxjitcompiler.so.${pkgver}"
    
    create_links

    rm -rf "${pkgdir}"/usr/{include,share,bin}
    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s lib32-nvidia-utils "${pkgdir}/usr/share/licenses/${pkgname}"
}
