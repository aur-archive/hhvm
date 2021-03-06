# Maintainer: Massimiliano Torromeo <massimiliano.torromeo@gmail.com>
# Contributor: James Miller <james@pocketrent.com>

pkgname=hhvm
pkgver=3.4.0
_thirdparty_commit=21bded6b6119ec24c53c4653868c05660529a62e
_folly_commit=32a9723ad4951fcc8b6324c55d967c3d2f21552e
_thrift_commit=0a455fe206fc4c32de8bf40caf71a75d03edf87c
_proxygen_commit=c4e89168873153ee74882f6d7bfadda16f91a308
pkgrel=1
pkgdesc="Virtual Machine, Runtime, and JIT for PHP"
arch=('x86_64')
url="http://hhvm.com"
license=('PHP')
depends=('boost-libs' 'google-glog' 'libmysqlclient' 'libmemcached' 'libzip' 'lz4'
         'libxslt' 'intel-tbb' 'libmcrypt' 'oniguruma' 'jemalloc' 'curl' 'libvpx'
         'libdwarf' 'imagemagick' 'libedit' 'sqlite' 'libyaml')
makedepends=('git' 'cmake' 'gcc' 'chrpath' 'boost' 'gflags' 'python2' 'pfff')
source=("https://github.com/facebook/hhvm/archive/HHVM-$pkgver.tar.gz"
        "hhvm-third-party-$_thirdparty_commit.tar.gz::https://github.com/hhvm/hhvm-third-party/archive/$_thirdparty_commit.tar.gz"
        "folly-$_folly_commit.tar.gz::https://github.com/facebook/folly/archive/$_folly_commit.tar.gz"
        "thrift-$_thrift_commit.tar.gz::https://github.com/facebook/fbthrift/archive/$_thrift_commit.tar.gz"
        "proxygen-$_proxygen_commit.tar.gz::https://github.com/facebook/proxygen/archive/$_proxygen_commit.tar.gz"
        'hhvm-max.patch'
        'hhvm.tmpfile'
        'hhvm.service'
        'hhvm@.service'
        'php.ini'
        'server.ini')
install=hhvm.install
backup=(etc/hhvm/{php,server}.ini)
options+=('!buildflags')

prepare() {
    cd "$srcdir"/$pkgname-HHVM-$pkgver
    patch -p1 -i "$srcdir"/hhvm-max.patch

    rm -rf third-party
    ln -s "$srcdir"/hhvm-third-party-$_thirdparty_commit third-party

    # no bundled pcre
    sed '/pcre/d' -i third-party/CMakeLists.txt

    cd third-party/folly
    rm -rf src
    ln -s "$srcdir"/folly-$_folly_commit src

    cd ../thrift
    rm -rf src
    ln -s "$srcdir"/fbthrift-$_thrift_commit src

    cd ../proxygen
    rm -rf src
    ln -s "$srcdir"/proxygen-$_proxygen_commit src
}

build() {
    cd "$srcdir"/$pkgname-HHVM-$pkgver
    msg2 "Building hhvm"

    # comment for tests
    HPHP_NOTEST=1 \
    cmake \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_PREFIX_PATH="$srcdir" \
        -DFREETYPE_INCLUDE_DIRS:PATH=/usr/include/freetype2 \
        .

    make

    for hacktool in hackificator remove_soft_types; do
        cd "$srcdir"/$pkgname-HHVM-$pkgver/hphp/hack/tools/$hacktool
        make depend
        make
    done
}

# check() {
#     cd "$srcdir"/$pkgname-HHVM-$pkgver/hphp/test
#     ./run --threads 8 quick
# }

package() {
    cd "$srcdir"/$pkgname-HHVM-$pkgver
    make DESTDIR="$pkgdir/" install
    rm -rf "$pkgdir"/usr/{lib*,include}

    cd hphp/hack/bin
    for bin in hh_* tools/*; do
        install -Dm755 $bin "$pkgdir"/usr/bin/$(basename $bin)
    done

    cd "$srcdir"
    install -Dm644 hhvm.tmpfile "$pkgdir"/usr/lib/tmpfiles.d/hhvm.conf
    install -Dm644 hhvm.service "$pkgdir"/usr/lib/systemd/system/hhvm.service
    install -Dm644 hhvm@.service "$pkgdir"/usr/lib/systemd/system/hhvm@.service

    install -Dm644 php.ini "$pkgdir"/etc/hhvm/php.ini
    install -Dm644 server.ini "$pkgdir"/etc/hhvm/server.ini
}

sha256sums=('65e194667722dd0f240321dce026e1707363ae42ce415a1974db5c28f54fb3ff'
            '55be18422577d97b9dfb1eaf34fe4fc8d348cb48fb3ae9190915d7dcba54fb8b'
            'e8ee31e3a08b4e42cd1bfe8c09a812d2887bb41b9ee0e67c8717ffdfd4d7c31b'
            'fa6cf805cb94c230fe20ba33d13eee76d6dc6956776c2bacd344cb698bab1d47'
            '0cc4fb29790305e2319182f5bf73ad6dd1f69e6f129e4efb7fff308e8aebb8dc'
            'ab98d74c382f503f1208407e891d26a88f9314fa2b631f6ec2a4a73ead644ba2'
            'c356010a6d6b976f387bb205a75ea07d5f40593a8010483f2ed0f66f112331bc'
            '8b50d1ef9f5f726e6d8d469a8c84d85ad63f8b507b97d258b4d751a0e3e221df'
            '59c640602929dac0aa34d06c668ed69361eb4b7b47a77f9aa0badb4d0b61571c'
            '3e3093f817706c238fad021483f114fd4ce0b45d84097dcb7870157fc9ec769f'
            '5b53bc57965e1c5151d720dc7f63f1b2e8ebd5e758b2ef0be3b74df38ebcbce0')
