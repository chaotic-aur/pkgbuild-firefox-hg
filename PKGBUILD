# Maintainer: Kyle De'Vir (QuartzDragon) <kyle[dot]devir[at]mykolab[dot]com>
# Contributor: Jonas Heinrich <onny@project-insanity.org>
# Contributor: Maxwell Anselm <silverhammermba+aur@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox-hg
_pkgname=firefox
pkgver=r445243.63eb34f9b171
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://www.mozilla.org/firefox/"
depends=(gtk3 mozilla-common libxt startup-notification mime-types dbus-glib
         ffmpeg nss hunspell-en_US sqlite ttf-font libpulse libvpx icu)
makedepends=(unzip zip diffutils python2-setuptools yasm mesa imake inetutils
             xorg-server-xvfb autoconf2.13 rust mercurial clang llvm jack gtk2
             python nodejs python2-psutil cbindgen)
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libnotify: Notification integration'
            'pulseaudio: Audio support'
            'speech-dispatcher: Text-to-Speech')
options=(!emptydirs !makeflags !strip)
_repo=https://hg.mozilla.org/mozilla-unified
conflicts=('firefox')
provides=('firefox')
source=('mozilla-unified::hg+https://hg.mozilla.org/mozilla-central/'
        $_pkgname.desktop
        firefox-symbolic.svg)
sha256sums=('SKIP'
            '677e1bde4c6b3cff114345c211805c7c43085038ca0505718a11e96432e9811a'
            '9a1a572dc88014882d54ba2d3079a1cf5b28fa03c5976ed2cb763c93dabbd797')

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

# Mozilla API keys (see https://location.services.mozilla.com/api)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact heftig@archlinux.org for
# more information.
_mozilla_api_key=16674381-f021-49de-8622-3021c5942aff

CFLAGS="-march=native -O2 -pipe -fstack-protector-strong -fno-plt"
CXXFLAGS="-march=native -O2 -pipe -fstack-protector-strong -fno-plt"

pkgver() {
  cd mozilla-unified
  printf "r%s.%s" "$(hg identify -n)" "$(hg identify -i)"
}

prepare() {
  cd mozilla-unified

  echo -n "$_google_api_key" >google-api-key
  echo -n "$_mozilla_api_key" >mozilla-api-key

  cat >.mozconfig <<END
ac_add_options --enable-application=browser

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-hardening
#ac_add_options --enable-optimize
ac_add_options --enable-rust-simd
ac_add_options --disable-dbus
ac_add_options --disable-necko-wifi

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux
export MOZILLA_OFFICIAL=1
export MOZ_TELEMETRY_REPORTING=1
export MOZ_REQUIRE_SIGNING=1

# Keys
ac_add_options --with-google-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-mozilla-api-keyfile=${PWD@Q}/mozilla-api-key

# System libraries
ac_add_options --enable-system-ffi
ac_add_options --enable-system-sqlite
ac_add_options --with-system-bz2
ac_add_options --with-system-icu
ac_add_options --with-system-jpeg
ac_add_options --with-system-libvpx
#ac_add_options --with-system-nspr
#ac_add_options --with-system-nss
ac_add_options --with-system-zlib
#ac_add_options --with-lto=full

# Features
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-startup-notification
ac_add_options --enable-crashreporter
ac_add_options --disable-gconf
ac_add_options --disable-updater
END
}

build() {
  cd mozilla-unified

  export CC="gcc"
  export CXX="g++"
  #export MOZ_PGO=1
  
  export MOZ_SOURCE_REPO="$_repo"
  export MOZ_NOSPAM=1
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"

  ./mach build
  ./mach buildsymbols
}

package() {
  cd mozilla-unified
  DESTDIR="$pkgdir" ./mach install
  find . -name '*crashreporter-symbols-full.zip' -exec cp -fvt "$startdir" {} +

  _vendorjs="$pkgdir/usr/lib/$_pkgname/browser/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);

// Opt all of us into e10s, instead of just 50%
pref("browser.tabs.remote.autostart", true);
END

  _distini="$pkgdir/usr/lib/$_pkgname/distribution/distribution.ini"
  install -Dm644 /dev/stdin "$_distini" <<END
[Global]
id=archlinux
version=1.0
about=Mozilla Firefox for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$_pkgname
app.partner.archlinux=archlinux
END

  for i in 16 22 24 32 48 64 128 256; do
    install -Dm644 browser/branding/official/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$_pkgname.png"
  done
  install -Dm644 browser/branding/official/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$_pkgname.png"
  install -Dm644 browser/branding/official/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/$_pkgname.png"
  install -Dm644 ../firefox-symbolic.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$_pkgname-symbolic.svg"

  install -Dm644 ../$_pkgname.desktop \
    "$pkgdir/usr/share/applications/$_pkgname.desktop"

  # Install a wrapper to avoid confusion about binary path
  install -Dm755 /dev/stdin "$pkgdir/usr/bin/$_pkgname" <<END
#!/bin/sh
exec /usr/lib/$_pkgname/firefox "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srf "$pkgdir/usr/bin/$_pkgname" \
    "$pkgdir/usr/lib/$_pkgname/firefox-bin"
}
