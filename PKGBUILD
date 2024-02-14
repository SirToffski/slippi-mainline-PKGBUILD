_projectname='slippi'
_mainpkgname="$_projectname-mainline"
pkgbase="$_mainpkgname-git"
pkgname=("$pkgbase")
pkgver=v4.0.0.mainline.beta.3.r2162.gdf9305c187
pkgrel=1
pkgdesc='https://slippi.gg/about'
arch=('x86_64')
url="https://github.com/project-slippi/dolphin"
license=('GPL2')
depends=('alsa-lib'
         'bluez-libs'
         'bzip2'
         'cubeb'
         'enet'
         'gcc-libs'
         'glibc'
         'hidapi'
         'libavcodec.so'
         'libavformat.so'
         'libavutil.so'
         'libcurl.so'
         'libevdev'
         'libfmt.so'
         'libgl'
         'libpulse'
         'libsfml-network.so'
         'libsfml-system.so'
         'libspng.so'
         'libswscale.so'
         'libudev.so'
         'libusb-1.0.so'
         'libx11'
         'libxi'
         'libxrandr'
         'libxxhash.so'
         'lzo'
         'mbedtls2'
         'pugixml'
         'qt6-base'
         'qt6-svg'
         'sfml'
         'speexdsp'
         'xz'
         'zlib-ng'
         'zstd'
)

makedepends=('cmake' 'git' 'miniupnpc' 'ninja' 'python' 'qt6-base' 'qt6-svg')
optdepends=('pulseaudio: PulseAudio backend')
options=('!lto')

commit='df9305c18797cd34c6c9bc2c6e63b4b1c4356b27'
#tag='v4.0.0-mainline-beta.3'

source=(
        "$pkgname::git+https://github.com/project-slippi/dolphin.git#commit=${commit}"
        #"$pkgname::git+https://github.com/project-slippi/dolphin.git#commit=${tag}"
        "$pkgname-implot::git+https://github.com/epezent/implot.git"
        "$pkgname-rcheevos::git+https://github.com/RetroAchievements/rcheevos.git"
        "$pkgname-corrosion::git+https://github.com/corrosion-rs/corrosion.git"
        "$pkgname-slippi-rust-extensions::git+https://github.com/project-slippi/slippi-rust-extensions.git"
        "$pkgname-vma::git+https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.git"
        "$pkgname-enet::git+https://github.com/lsalzman/enet.git"
)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
)

_sourcedirectory="$pkgname"
_dolphinemu="dolphin-emu"

prepare() {
        cd "$srcdir/$_sourcedirectory/"
        if [ -d 'build/' ]; then rm -rf 'build/'; fi
        mkdir 'build/'

        # Provide submodules
        declare -A _submodules=(
                [enet]='enet/enet'
                [implot]='implot/implot'
                [rcheevos]='rcheevos/rcheevos'
                [vma]='VulkanMemoryAllocator'
                [corrosion]='corrosion'
                [slippi-rust-extensions]='SlippiRustExtensions'
        )

        for _submod in "${!_submodules[@]}"; do
                _path="Externals/${_submodules[$_submod]}"
                git submodule init "$_path"
                git config "submodule.$_path.url" "$srcdir/$pkgname-$_submod/"
                git -c protocol.file.allow=always submodule update "$_path"
        done

}

pkgver() {
        cd "$srcdir/$_sourcedirectory/"
        git describe --long --tags | sed -e 's/-\([^-]*-g[^-]*\)$/-r\1/' -e 's/-/./g'
}

build() {


        export LDFLAGS="-Wl,-O1,--sort-common,--as-needed,--copy-dt-needed-entries,-z,relro,-z,now"
        export CFLAGS+=' -Wno-error=nonnull -I/usr/include/mbedtls2'
        export CXXFLAGS+=' -Wno-error=nonnull -I/usr/include/mbedtls2'
        export LDFLAGS+=' -L/usr/lib/mbedtls2'

        CMAKE_FLAGS='-DLINUX_LOCAL_DEV=true -DSLIPPI_PLAYBACK=false -DUSE_SYSTEM_LIBS=ON -DENABLE_TESTS=OFF -DENABLE_NOGUI=OFF -DUSE_SYSTEM_ENET=OFF -DENABLE_CLI_TOOL=OFF -DUSE_SYSTEM_LIBMGBA=OFF -DUSE_SYSTEM_MINIZIP=OFF -Wno-dev'

        cd "$srcdir/$_sourcedirectory/"
        mkdir -p build
        cd build
        cmake ${CMAKE_FLAGS} ../
        cmake --build . --target dolphin-emu -- -j"$(nproc)"

}

package() {
        pkgdesc="$pkgdesc"
        provides=("$_mainpkgname")
        conflicts=("$_mainpkgname" "slippi-online-git")

        cd "$srcdir/$_sourcedirectory/"
        make DESTDIR="$pkgdir" -C 'build/' install
        mv "$pkgdir/usr/local/bin/$_dolphinemu" "$pkgdir/usr/local/bin/$_mainpkgname"
        cp -r "Data/Sys/" "$pkgdir/usr/local/bin/"
        rm -r "$pkgdir/usr/local/share/man/"
        install -Dm644 'build/cargo/build/x86_64-unknown-linux-gnu/release/libslippi_rust_extensions.so' "$pkgdir/usr/lib/libslippi_rust_extensions.so"
        rm -rf "$pkgdir/usr/include"
        rm -rf "$pkgdir/usr/lib/libdiscord-rpc.a"


}