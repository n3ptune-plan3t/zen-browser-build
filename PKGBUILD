# Maintainer: envolution
# Contributor: Gavin Luo <lunt.luo@gmail.com>
# shellcheck shell=bash disable=SC2034,SC2154
# ci|prebuild=setffver.sh| https://github.com/envolution/aur/blob/main/maintain/build/zen-browser/setffver.sh

pkgbase=zen-browser
pkgname=("$pkgbase")
pkgver=1.14.11b
_zen_version=${pkgver//_/-}
_firefox_version=141.0.2
pkgrel=1
pkgdesc='Experience tranquillity while browsing the web without people tracking you'
url='https://zen-browser.app/'
arch=('x86_64')
license=(MPL-2.0)
depends=(
  alsa-lib
  at-spi2-core
  bash
  cairo
  dbus
  ffmpeg
  fontconfig
  freetype2
  gcc-libs
  gdk-pixbuf2
  glib2
  glibc
  gtk3
  hicolor-icon-theme
  libpulse
  libx11
  libxcb
  libxcomposite
  libxdamage
  libxext
  libxfixes
  libxrandr
  libxss
  libxt
  mime-types
  nspr
  nss
  pango
  ttf-font)
makedepends=(
  git
  pnpm
  rsync
  cbindgen
  clang
  diffutils
  imake
  lld
  llvm
  mesa
  nasm
  nodejs-lts-iron
  #  pyenv
  python
  rust
  unzip
  wasi-compiler-rt
  wasi-libc
  wasi-libc++
  wasi-libc++abi
  xorg-server-xvfb
  yasm
  zip)
optdepends=(
  'hunspell-en_US: Spell checking, American English'
  'libnotify: Notification integration'
  'networkmanager: Location detection via available WiFi networks'
  'speech-dispatcher: Text-to-Speech'
  'xdg-desktop-portal: Screensharing with Wayland')
options=(
  !emptydirs
  !lto
  !makeflags
  !debug)

_repo='https://github.com/zen-browser/desktop'
source=(
  "git+$_repo.git#tag=$_zen_version"
  "https://archive.mozilla.org/pub/firefox/releases/${_firefox_version}/source/firefox-${_firefox_version}.source.tar.xz"
  "firefox-l10n::git+https://github.com/mozilla-l10n/firefox-l10n"
  zen-browser.desktop
  0003-do-not-disable-system-extensions.zen.patch
  0004-fix-package-json.zen.patch
  0005-source-firefox-language-packs.patch
  )
sha256sums=('e717dd50d5940ad65506cc44d78f3727d28f1278b08f038cdf2082adcf161476'
            'c33937fe2f6ad29af3de8f1a128c054afbd64821f702bf98d9f4079b97d37f3a'
            'SKIP'
            '523fba56892357a1b37811021e06d548cb94af58948294a436c566581e7454a9'
            '36bff2af04da55da0cc71f960d921889ccf21c11fcd8343087c144dfcc50f10a'
            '803c3f456abfc1acd963b594cf684aed2453534e7ab951abc38efa0351b648a1'
            '7702c197f5509e4ec7e744f74105cc8339b631e37f7a0b41bf3070bf3ccc92a7')
noextract=("firefox-${_firefox_version}.source.tar.xz")

_languages=(zh-CN zh-TW ja)

for _lang in "${_languages[@]}"; do
  _pkgname=$pkgbase-i18n-${_lang,,}
  pkgname+=("$_pkgname")

  eval "package_$_pkgname() {
  arch=(any)
  depends=('$pkgbase>=$pkgver')
  optdepends=()
  pkgdesc='Language pack for Zen Browser ($_lang)'

  _package_i18n '$_lang'
}"
done

_package_i18n() {
  msg2 "install langpack $1"
  install -Dvm644 \
    "$srcdir/$_OBJ_DIR/dist/linux-$CARCH/xpi/zen-$_firefox_version.$1.langpack.xpi" \
    "$pkgdir/usr/lib/$pkgbase/browser/extensions/langpack-$1@firefox.mozilla.org.xpi"
}

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

_OBJ_DIR=obj
prepare() {
  cd desktop

  msg2 "init repo submodules"
  git submodule init
  git config submodule.l10n.url "$srcdir/l10n"
  git -c protocol.file.allow=always submodule update

  # Apply patches
  msg2 "apply patches"
  git apply -3 "$srcdir"/*.zen.patch
  patch -Np1 -i ../0005-source-firefox-language-packs.patch

  msg2 "prepare dependencies"
  pnpm config set store-dir "$srcdir"/pnpm-store
  pnpm install

  msg2 "prepare surfer build environment"
  pnpm surfer ci --brand release --display-version "$_zen_version"
  # setup Firefox source
  install -Dvm644 "$srcdir/firefox-${_firefox_version}.source.tar.xz" -t "./.surfer/engine"
  pnpm surfer download
  # Import patches into the firefox
  env SURFER_COMPAT="$CARCH" pnpm surfer import

  msg2 "prepare firefox l10n"
  srcdir="$srcdir" sh scripts/download-language-packs.sh

  msg2 "prepare custom mozconfig"
  echo -n "$_google_api_key" >google-api-key
  cat >mozconfig <<END
# # sccache
# mk_add_options 'export RUSTC_WRAPPER=sccache'
# mk_add_options 'export CCACHE_CPP2=yes'
# ac_add_options --with-ccache=sccache

# ac_add_options --enable-application=browser
# Incompatible with surfer, disable this configuration
mk_add_options MOZ_OBJDIR=${srcdir}/$_OBJ_DIR

ac_add_options --prefix=/usr
# ac_add_options --enable-release
# ac_add_options --enable-hardening
# ac_add_options --enable-optimize
# ac_add_options --enable-rust-simd
ac_add_options --enable-linker=lld
# ac_add_options --disable-install-strip
# ac_add_options --disable-elf-hack
# It seems to be overwritten by surfer internal mozconfg, let's keep it for now
ac_add_options --disable-bootstrap
ac_add_options --with-wasi-sysroot=/usr/share/wasi-sysroot

# Branding
# ac_add_options --enable-official-branding
# ac_add_options --enable-update-channel=release
# ac_add_options --with-distribution-id=org.archlinux
# ac_add_options --with-unsigned-addon-scopes=app,system
ac_add_options --allow-addon-sideload
# export MOZILLA_OFFICIAL=1
export MOZ_APP_REMOTINGNAME=$pkgbase

# Keys
ac_add_options --with-google-location-service-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-google-safebrowsing-api-keyfile=${PWD@Q}/google-api-key

# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# Features
# ac_add_options --enable-alsa
# ac_add_options --enable-jack
# ac_add_options --enable-crashreporter
ac_add_options --disable-crashreporter
ac_add_options --disable-updater
# ac_add_options --disable-tests
END

}

_mach() {
  "$srcdir/desktop/engine/mach" "$@"
}
build() {
  export MACH_BUILD_PYTHON_NATIVE_PACKAGE_SOURCE=pip
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  MOZ_BUILD_DATE="$(date -u${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH} +%Y%m%d%H%M%S)"
  export MOZ_BUILD_DATE
  # export MOZ_BUILD_PRIORITY=normal
  export MOZ_NOSPAM=1

  # malloc_usable_size is used in various parts of the codebase
  CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"

  # Breaks compilation since https://bugzilla.mozilla.org/show_bug.cgi?id=1896066
  CFLAGS="${CFLAGS/-fexceptions/}"
  CXXFLAGS="${CXXFLAGS/-fexceptions/}"
  LD=ld.lld
  RUSTFLAGS="-Clink-arg=-fuse-ld=lld"
  # LTO needs more open files
  ulimit -n 4096

  # The Python version in the official repository is too high
  # export PYENV_ROOT="$srcdir/.pyenv"
  # export PATH="$PYENV_ROOT/bin:$PATH"
  # export PYENV_VERSION=3.11
  # pyenv install $PYENV_VERSION
  # eval "$(pyenv init -)"

  # Run PGO Build
  msg2 "build zen browser"
  (
    cd "$srcdir/desktop"

    # The .so dependencies of clang-plguin are packaged into libclang-cpp.so in Arch Linux
    # so we need this patch
    sed -i 's/clangASTMatchers/clang-cpp/g' ./engine/build/clang-plugin/moz.build

    pnpm run ffprefs
    env \
      SURFER_COMPAT="$CARCH" \
      SURFER_PLATFORM=linux \
      ZEN_RELEASE_BRANCH=release \
      ZEN_RELEASE=1 \
      LLVM_PROFDATA=llvm-profdata \
      dbus-run-session \
      xvfb-run -s "-screen 0 1920x1080x24 -nolisten local" \
      pnpm surfer build --skip-patch-check
  )

  # Build langpacks
  echo $_firefox_version >"$srcdir/desktop/engine/browser/config/version.txt"
  for _lang in "${_languages[@]}"; do
    msg2 "build langpack: ${_lang}"
    _mach build "merge-$_lang"
    _mach build "langpack-$_lang"
  done
}

package_zen-browser() {
  provides=("$pkgbase=$pkgver")
  replace=("$pkgbase-generic" "$pkgbase-specific")

  # package variant
  msg2 "install to pkgdir"
  DESTDIR="$pkgdir" _mach install

  local _appdir="$pkgdir/usr/lib/$pkgbase"

  msg2 "zen -> $pkgbase"
  rm -rf "$_appdir"
  mv "$pkgdir"/usr/lib/zen "$_appdir"

  # Replace duplicate binary
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  msg2 "replace duplicate zen binary"
  ln -srvf "$_appdir"/zen "$_appdir/zen-bin"
  ln -srvf "$_appdir"/zen "$pkgdir"/usr/bin/zen

  msg2 "install distribution config files from Zen Browser"
  install -Dvm644 "$srcdir"/desktop/build/AppDir/distribution/*.json -t "$_appdir/distribution"

  msg2 "install vendor.js"
  local _vendorjs="$_appdir/browser/defaults/preferences/vendor.js"
  install -Dvm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Don't disable extensions in the application directory
pref("extensions.autoDisableScopes", 11);

// TODO: Enable GNOME Shell search provider
// pref("browser.gnome-search-provider.enabled", true);
END

  msg2 "install distribution.ini"
  local _distini="$_appdir/distribution/distribution.ini"
  install -Dvm644 /dev/stdin "$_distini" <<END
[Global]
id=archlinux
version=1.0
about=Zen Browser for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$pkgbase
app.partner.archlinux=archlinux
END

  msg2 "install icons"
  for i in 16 32 48 64 128; do
    install -d "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps"
    ln -srvf \
      "$_appdir/browser/chrome/icons/default/default${i}.png" \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgbase.png"
  done
  install -Dm0644 "$srcdir"/desktop/docs/assets/zen-black.svg "$pkgdir/usr/share/icons/hicolor/scalable/apps/$pkgbase.svg"
  install -Dvm644 "$srcdir"/desktop/docs/assets/zen-black.svg "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$pkgbase-symbolic.svg"

  msg2 "install desktop file"
  #install -Dvm644 "$srcdir"/desktop/build/AppDir/*.desktop "$pkgdir/usr/share/applications/$pkgbase.desktop"
  install -Dvm644 "$srcdir"/zen-browser.desktop "$pkgdir/usr/share/applications/$pkgbase.desktop"

  msg2 "use system certificates"
  local _nssckbi="$_appdir/libnssckbi.so"
  if test -e "$_nssckbi"; then
    ln -srvf "$pkgdir/usr/lib/libnssckbi.so" "$_nssckbi"
  fi

  # TODO: Enable GNOME Shell search provider
  #   local sprovider="$pkgdir/usr/share/gnome-shell/search-providers/$pkgbase.search-provider.ini"
  #   install -Dvm644 /dev/stdin "$sprovider" <<END
  # [Shell Search Provider]
  # DesktopId=$pkgbase.desktop
  # BusName=org.mozilla.${pkgname//-/_}.SearchProvider
  # ObjectPath=/org/mozilla/${pkgname//-/_}/SearchProvider
  # Version=2
  # END
}
# vim:set ts=2 sw=2 et:
