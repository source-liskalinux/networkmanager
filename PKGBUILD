# PKGBUILD For pam

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=pam
pkgver=1.7.2
pkgrel=1
pkgdesc="A flexible mechanism for authenticating user (Pluggable Authentication Modules)"
arch=('x86_64')
url="https://github.com/source-liskalinux/pam"
license=('GPL-2.0-or-later' 'BSD-3-Clause')
depends=('glibc' 'libxcrypt')
makedepends=('meson' 'ninja')
provides=('libpam' 'libpam.so' 'libpam_misc' 'libpamc')
source=("https://github.com/linux-pam/linux-pam/releases/download/v${pkgver}/Linux-PAM-${pkgver}.tar.xz")
sha256sums=('SKIP')

build() {
    cd "${srcdir}/Linux-PAM-${pkgver}"
    meson setup build \
        --prefix=/usr \
        --sysconfdir=/etc \
        --libdir=/usr/lib \
        --mandir=/usr/share/man \
        -Ddocs=disabled \
        -Dexamples=false \
        -Dselinux=disabled \
        -Daudit=disabled \
        -Deconf=disabled \
        -Dnis=disabled \
        -Ddb=auto
    meson compile -C build
}

package() {
    cd "${srcdir}/Linux-PAM-${pkgver}"
    DESTDIR="${pkgdir}" meson install -C build
    # securedir defaults to $(libdir)/security = /usr/lib/security, same
    # convention already used by elogind's -Dpamlibdir=/usr/lib/security
    # for pam_elogind.so, so both land in the same module directory.
    if [ ! -f "${pkgdir}/usr/lib/security/pam_unix.so" ]; then
        echo "ERROR: pam_unix.so not found after install! Build/install failed silently." >&2
        exit 1
    fi
    install -d -m755 "${pkgdir}/etc/pam.d"
    # Mandatory fallback service. If a program asks PAM for a service name
    # with no matching /etc/pam.d/<service> file, PAM falls through to this
    # one - deny-by-default is the standard safe posture, and Linux-PAM's
    # own test suite requires this exact file to exist.
    cat > "${pkgdir}/etc/pam.d/other" <<'EOF'
#%PAM-1.0
auth      required   pam_deny.so
account   required   pam_deny.so
password  required   pam_deny.so
session   required   pam_deny.so
EOF
    # Shared base other service files can "include". pam_securetty is left
    # out here (it belongs on login specifically, not su/passwd).
    cat > "${pkgdir}/etc/pam.d/system-auth" <<'EOF'
#%PAM-1.0
auth      required   pam_unix.so try_first_pass nullok
account   required   pam_unix.so
password  required   pam_unix.so try_first_pass nullok sha512 shadow
session   required   pam_unix.so
EOF
    # Used by shadow-utils' login (busybox getty execs this). pam_securetty
    # restricts root logins to ttys listed in /etc/securetty (shipped by
    # the filesystem package). pam_elogind registers the session with
    # elogind's seat0 so loginctl/session tracking works without systemd.
    cat > "${pkgdir}/etc/pam.d/login" <<'EOF'
#%PAM-1.0
auth       required     pam_securetty.so
auth       include      system-auth
account    include      system-auth
password   include      system-auth
session    include      system-auth
session    optional     pam_elogind.so
session    optional     pam_lastlog.so
EOF
    cat > "${pkgdir}/etc/pam.d/su" <<'EOF'
#%PAM-1.0
auth      include   system-auth
account   include   system-auth
password  include   system-auth
session   include   system-auth
EOF
    cat > "${pkgdir}/etc/pam.d/passwd" <<'EOF'
#%PAM-1.0
password  include   system-auth
EOF
    rm -rf "${pkgdir}/usr/lib/systemd"
    rm -rf "${pkgdir}/lib/systemd"
}
