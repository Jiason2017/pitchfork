include ../../../mk/pitchfork.mk

# Local variables
_NAME   = libressl-2.5.5
_URL    = http://ftp.openbsd.org/pub/OpenBSD/LibreSSL
_WRKSRC = $(WORKDIR)/$(_NAME)
_CFLAGS = $(CFLAGS)
_FILE   = $(_NAME).tar.gz

# Local works
ifneq ($(origin HAVE_OPENSSL),undefined)
    HAVE_LIBSSL = $(HAVE_OPENSSL)
endif
ifeq ($(origin HAVE_LIBSSL),undefined)
do-extract: $(_WRKSRC)
$(_WRKSRC): | do-fetch
	tar zxf $(_FILE) -C $(WORKDIR)
do-fetch: $(_FILE)
$(_FILE):
	$(PFHOME)/bin/pitchfork fetch --url $(_URL)/$@
do-config: $(_WRKSRC)/config.log
$(_WRKSRC)/config.log: | do-extract
	(cd $(_WRKSRC) && CFLAGS="$(_CFLAGS)" ./configure --prefix=$(STAGING)/$(_NAME)) >& \
        config.log || $(PFHOME)/bin/diewith config.log
do-build: $(_WRKSRC)/ssl/.libs/libssl.$(DYLIB)
$(_WRKSRC)/ssl/.libs/libssl.$(DYLIB): | do-config
	$(MAKE) -C $(_WRKSRC) >& build.log || $(PFHOME)/bin/diewith build.log
do-install: $(PREFIX)/var/pkg/$(_NAME)
$(PREFIX)/var/pkg/$(_NAME): | do-build
	rm -rf $(STAGING)/$(_NAME)
	$(MAKE) -C $(_WRKSRC) install >& install.log || $(PFHOME)/bin/diewith install.log
	$(PERL) -pi -e 's[$(STAGING)/$(_NAME)][$(PREFIX)]g' $(STAGING)/$(_NAME)/lib/pkgconfig/*.pc
	cd $(STAGING)/$(_NAME) && tar cf - * | tar xf - -C $(PREFIX)
	find $(STAGING)/$(_NAME) ! -type d|awk -F '$(STAGING)/$(_NAME)/' '{print $$2}' > $@
else
ifneq ($(wildcard $(HAVE_LIBSSL)/lib/libssl*),)
    _SSLLIB = $(HAVE_LIBSSL)/lib
else ifneq ($(wildcard $(HAVE_LIBSSL)/lib64/libssl*),)
    _SSLLIB = $(HAVE_LIBSSL)/lib64
else
    $(error $(HAVE_LIBSSL)/lib/libssl* not found)
endif
do-install: $(PREFIX)/var/pkg/$(_NAME)
$(PREFIX)/var/pkg/$(_NAME):
	rm -rf $(STAGING)/$(_NAME)
	mkdir -p \
                 $(STAGING)/$(_NAME)/share/man/man1 \
                 $(STAGING)/$(_NAME)/include \
                 $(STAGING)/$(_NAME)/lib/pkgconfig \
                 $(STAGING)/$(_NAME)/etc
	test -d $(_SSLLIB)/pkgconfig && cp -aL $(_SSLLIB)/pkgconfig/libcrypto.pc $(STAGING)/$(_NAME)/lib/pkgconfig/ || true
	test -d $(_SSLLIB)/pkgconfig && cp -aL $(_SSLLIB)/pkgconfig/*ssl.pc      $(STAGING)/$(_NAME)/lib/pkgconfig/ || true
	#cp -aL $(_SSLLIB)/pkgconfig/libtls.pc    $(STAGING)/$(_NAME)/lib/pkgconfig/
	cp -aL $(_SSLLIB)/libcrypto*             $(STAGING)/$(_NAME)/lib/
	cp -aL $(_SSLLIB)/libssl*                $(STAGING)/$(_NAME)/lib/
	#cp -aL $(_SSLLIB)/libtls*                $(STAGING)/$(_NAME)/lib/
	cp -aL $(HAVE_LIBSSL)/include/openssl            $(STAGING)/$(_NAME)/include/
	#cp -aL $(HAVE_LIBSSL)/include/tls.h              $(STAGING)/$(_NAME)/include/
	#cp -aL $(HAVE_LIBSSL)/share/man/man1/openssl.1   $(STAGING)/$(_NAME)/share/man/man1/
	cd $(STAGING)/$(_NAME) && tar cf - * | tar xf - -C $(PREFIX)
	find $(STAGING)/$(_NAME) ! -type d|awk -F '$(STAGING)/$(_NAME)/' '{print $$2}' > $@
endif
do-clean:
	$(MAKE) -C $(_WRKSRC) clean
	rm -f build.log install.log
	rm -rf $(STAGING)/$(_NAME)
do-distclean:
	$(MAKE) -C $(_WRKSRC) distclean
	rm -f $(_WRKSRC)/config.log config.log
do-flush:
	rm -rf $(_WRKSRC) || true
