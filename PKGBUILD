# Maintainer: 0xyousif

pkgbase=linux-yousif
pkgname=(
    linux-yousif
    linux-yousif-headers
)
pkgver=7.2.8
pkgrel=1
pkgdesc='Custom Linux kernel for x86-64-v4'
arch=('x86_64')
url='https://www.kernel.org/'
license=('GPL-2.0-only')
makedepends=(
    'bc'
    'binutils'
    'bison'
    'cpio'
    'gettext'
    'libelf'
    'openssl'
    'pahole'
    'perl'
    'python'
    'rust'
    'rust-bindgen'
    'rust-src'
    'tar'
    'xxhash'
    'xz'
    'zlib'
    'zstd'
    'clang'
    'llvm'
    'lld'
)
options=('!strip')

source=(
    "https://cdn.kernel.org/pub/linux/kernel/v7.x/linux-${pkgver}.tar.xz"
    '.config'
)

sha256sums=(
    'SKIP'
    'SKIP'
)

prepare() {
    cd "linux-${pkgver}"

    cp "${srcdir}/.config" .config

    scripts/config \
        -e GENERIC_CPU \
        -d MZEN4 \
        -d X86_NATIVE_CPU \
        --set-val X86_64_VERSION 4

    scripts/config \
        -d CC_OPTIMIZE_FOR_PERFORMANCE \
        -e CC_OPTIMIZE_FOR_PERFORMANCE_O3

    scripts/config \
        -e LTO_CLANG_THIN

    make \
        LLVM=1 \
        LLVM_IAS=1 \
        olddefconfig

    echo "-${pkgrel}" > localversion.10-pkgrel
    echo "-${pkgbase#linux-}" > localversion.20-pkgname

    make -s LLVM=1 kernelrelease > version
}

build() {
    cd "linux-${pkgver}"

    make \
        CC=clang \
        LD=ld.lld \
        LLVM=1 \
        LLVM_IAS=1 \
        -j"$(nproc)" \
        all
}

_package() {
    pkgdesc="The ${pkgdesc} kernel and modules"
    provides=('linux' 'KMOD')
    depends=(
        'coreutils'
        'kmod'
        'initramfs'
    )
    optdepends=(
        'linux-firmware: firmware images needed for some devices'
        'wireless-regdb: to set the correct wireless channels of your country'
    )

    cd "linux-${pkgver}"

    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    install -Dm644 \
        "$(make -s LLVM=1 image_name)" \
        "$modulesdir/vmlinuz"

    echo "$pkgbase" |
        install -Dm644 /dev/stdin \
        "$modulesdir/pkgbase"

    ZSTD_CLEVEL=19 make \
        CC=clang \
        LD=ld.lld \
        LLVM=1 \
        LLVM_IAS=1 \
        INSTALL_MOD_PATH="$pkgdir/usr" \
        INSTALL_MOD_STRIP=1 \
        DEPMOD=/doesnt/exist \
        modules_install

    rm -f "$modulesdir/build"
}

_package-headers() {
    pkgdesc="Headers and scripts for building modules for the ${pkgdesc} kernel"
    provides=('linux-headers')
    depends=(
        'binutils'
        'glibc'
        'libelf'
        'libgcc'
        'openssl'
        'pahole'
        'xxhash'
        'zlib'
        'zstd'
    )

    cd "linux-${pkgver}"

    local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

    install -Dt "$builddir" -m644 \
        .config \
        Makefile \
        Module.symvers \
        System.map \
        localversion.* \
        version \
        vmlinux

    install -Dt "$builddir/kernel" -m644 \
        kernel/Makefile

    install -Dt "$builddir/arch/x86" -m644 \
        arch/x86/Makefile

    cp -t "$builddir" -a scripts

    ln -srt "$builddir" \
        "$builddir/scripts/gdb/vmlinux-gdb.py"

    install -Dt "$builddir/tools/objtool" \
        tools/objtool/objtool

    if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
        install -Dt "$builddir/tools/bpf/resolve_btfids" \
            tools/bpf/resolve_btfids/resolve_btfids
    fi

    cp -t "$builddir" -a include
    cp -t "$builddir/arch/x86" -a arch/x86/include

    install -Dt "$builddir/arch/x86/kernel" -m644 \
        arch/x86/kernel/asm-offsets.s

    install -Dt "$builddir/drivers/md" -m644 \
        drivers/md/*.h

    install -Dt "$builddir/net/mac80211" -m644 \
        net/mac80211/*.h

    if compgen -G 'drivers/media/i2c/msp3400-driver.h' >/dev/null; then
        install -Dt "$builddir/drivers/media/i2c" -m644 \
            drivers/media/i2c/msp3400-driver.h
    fi

    if compgen -G 'drivers/media/usb/dvb-usb/*.h' >/dev/null; then
        install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 \
            drivers/media/usb/dvb-usb/*.h
    fi

    if compgen -G 'drivers/media/dvb-frontends/*.h' >/dev/null; then
        install -Dt "$builddir/drivers/media/dvb-frontends" -m644 \
            drivers/media/dvb-frontends/*.h
    fi

    if compgen -G 'drivers/media/tuners/*.h' >/dev/null; then
        install -Dt "$builddir/drivers/media/tuners" -m644 \
            drivers/media/tuners/*.h
    fi

    find . -name 'Kconfig*' \
        -exec install -Dm644 {} "$builddir/{}" \;

    if compgen -G 'rust/*.rmeta' >/dev/null; then
        install -Dt "$builddir/rust" -m644 \
            rust/*.rmeta
    fi

    if compgen -G 'rust/*.so' >/dev/null; then
        install -Dt "$builddir/rust" -m644 \
            rust/*.so
    fi

    make \
        INSTALL_MOD_PATH="$pkgdir/usr" \
        vdso_install \
        link=

    for arch in "$builddir"/arch/*/; do
        [[ "$arch" == */x86/ ]] && continue
        rm -rf "$arch"
    done

    rm -rf "$builddir/Documentation"

    find -L "$builddir" \
        -type l \
        -delete

    find "$builddir" \
        -type f \
        -name '*.o' \
        -delete

    mkdir -p "$pkgdir/usr/src"

    ln -sr \
        "$builddir" \
        "$pkgdir/usr/src/$pkgbase"
}

package_linux-yousif() {
    _package
}

package_linux-yousif-headers() {
    _package-headers
}
