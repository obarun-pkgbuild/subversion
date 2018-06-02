# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://git.archlinux.org/svntogit/packages.git/log/trunk?h=packages/subversion
# 						Maintainer: Angel Velasquez <angvp@archlinux.org>
# 						Maintainer: Felix Yan <felixonmars@archlinux.org>
# 						Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# 						Contributor: Paul Mattal <paul@archlinux.org>
# 						Contributor: Jason Chu <jason@archlinux.org>

pkgname=subversion
pkgver=1.10.0
pkgrel=3
pkgdesc="A Modern Concurrent Version Control System"
arch=(x86_64)
url="http://subversion.apache.org/"
license=('APACHE')
depends=('file' 'libutf8proc' 'serf' 'sqlite')
makedepends=('apache' 'python2' 'perl' 'swig' 'java-environment<10'
             'libgnome-keyring' 'kwallet' 'kdelibs4support' 'ruby')
optdepends=('libgnome-keyring: for GNOME Keyring for auth credentials'
            'kwallet: for KWallet for auth credentials'
            'bash-completion: for svn bash completion'
            'python2: for some hook scripts'
            'java-environment: for Java support'
            'ruby: for some hook scripts')
checkdepends=('ruby-test-unit')
provides=('svn')
backup=('etc/xinetd.d/svn' 'etc/conf.d/svnserve')
options=('!makeflags' '!emptydirs')
source=(https://www.apache.org/dist/subversion/subversion-${pkgver}.tar.bz2
        svn
        svnserve.conf
        svnserve.tmpfiles
        subversion.rpath.fix.patch
        ruby-frozen-nil.patch)
sha512sums=('26e49c961d54c4e78067caa202f384c36544e0ce7506abbffe73e59186ac34559a70cbd6c0934ed47dcc5c3580fce0f0cc2ef14b77695c26e8dbbc6358d1ec6a'
            '3df59e92aa0314ff6adce26e2e1162bf2872ca03ff1f78891081a60e67b521b6046b4a2f85f718dcd27f9d5709594658817a09548cdb74e3976d371dbe47e7db'
            'f7f2ceac2446cc94ac2be3404083cc54a0f1f4d04d5301f600dfafca38819669bcffdfa45f1b90b9f3cdb042469385a764f11dc1a827f10c23ddf73b7ac6c9da'
            '7775f4da5003970c9ebdc2f696ba090df194a77d9daed791875488c943f72ae496b5f9cc6f3ff9f3f4de9f352a3b518137babdea38947d1a2d5dd16aa1844036'
            '60d538160e738eb3b3e84a3881fe5a8d75c79053d3f31c4c29ef6ace6ccc5dd4367ed712766c911bae3436e9706e4dd144b270bb45161a6c1834a37e154d0440'
            'bb772e55acd9601121ad06b254c364e8d8cf772ca59b8df0cf4c5c5ecba110d4108d0363672f121f770550cdd052802474029e57643258f398aacd2b63ccb898')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal <eric@obarun.org>

prepare() {
   cd ${pkgname}-${pkgver}
   patch -Np0 -i ../subversion.rpath.fix.patch
   patch -p1 -i ../ruby-frozen-nil.patch
   sed -i 's|/usr/bin/env python|/usr/bin/env python2|' tools/hook-scripts/{,mailer/{,tests/}}*.py subversion/tests/cmdline/*.py
}

build() {
   cd ${pkgname}-${pkgver}
   export PYTHON=/usr/bin/python2
   ./configure 	--prefix=/usr \
				--with-apr=/usr \
				--with-apr-util=/usr \
				--with-zlib=/usr \
				--with-serf=/usr \
				--with-apxs \
				--with-sqlite=/usr \
				--enable-javahl \
				--with-jdk=/usr/lib/jvm/default \
				--with-gnome-keyring \
				--with-kwallet \
				--with-apache-libexecdir=/usr/lib/httpd/modules \
				--with-ruby-sitedir=/usr/lib/ruby/vendor_ruby \
				--disable-static

   make LT_LDFLAGS="-L$Fdestdir/usr/lib"
   make swig_pydir=/usr/lib/python2.7/site-packages/libsvn \
     swig_pydir_extra=/usr/lib/python2.7/site-packages/svn swig-py swig-pl javahl swig-rb
}

check() {
   cd ${pkgname}-${pkgver}
   export LANG=C LC_ALL=C
   make check check-swig-pl check-swig-py check-swig-rb CLEANUP=yes # check-javahl
}

package() {
   cd ${pkgname}-${pkgver}

   export LD_LIBRARY_PATH="${pkgdir}"/usr/lib:${LD_LIBRARY_PATH}
   make DESTDIR="${pkgdir}" INSTALLDIRS=vendor \
     swig_pydir=/usr/lib/python2.7/site-packages/libsvn \
     swig_pydir_extra=/usr/lib/python2.7/site-packages/svn \
     install install-swig-py install-swig-pl install-javahl install-swig-rb

   install -dm755 "${pkgdir}"/usr/share/subversion
   cp -a tools/hook-scripts "${pkgdir}"/usr/share/subversion/
   rm "${pkgdir}"/usr/share/subversion/hook-scripts/*.in

   ## svnserve ...

   # xinetd
   install -D -m 644 "${srcdir}"/svn "${pkgdir}"/etc/xinetd.d/svn

   # ... pacopts applytmp
   install -D -m 644 "${srcdir}"/svnserve.tmpfiles "${pkgdir}"/usr/lib/tmpfiles.d/svnserve.conf

   # ... common config
   install -D -m 644 "${srcdir}"/svnserve.conf "${pkgdir}"/etc/conf.d/svnserve

   install -Dm 644 tools/client-side/bash_completion \
     "${pkgdir}"/usr/share/bash-completion/completions/subversion
   for i in svn svnadmin svndumpfilter svnlook svnsync svnversion; do
      ln -sf subversion "${pkgdir}"/usr/share/bash-completion/completions/${i}
   done
}

