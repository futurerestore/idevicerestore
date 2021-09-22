AM_CFLAGS = \
	$(GLOBAL_CFLAGS) \
	$(LFS_CFLAGS) \
	$(libirecovery_CFLAGS) \
	$(libimobiledevice_CFLAGS) \
	$(libplist_CFLAGS) \
	$(limd_glue_CFLAGS) \
	$(libzip_CFLAGS) \
	$(zlib_CFLAGS) \
	$(openssl_CFLAGS) \
	$(libcurl_CFLAGS)

AM_LDFLAGS = \
	$(AC_LDFLAGS) \
	$(libirecovery_LIBS) \
	$(libimobiledevice_LIBS) \
	$(libplist_LIBS) \
	$(limd_glue_LIBS) \
	$(libzip_LIBS) \
	$(zlib_LIBS) \
	$(openssl_LIBS) \
	$(libcurl_LIBS)

AM_LDADD = $(AC_LDADD)

if NOMAIN
noinst_LTLIBRARIES = libidevicerestore.la
libidevicerestore_la_SOURCES = idevicerestore.c common.c tss.c fls.c mbn.c img3.c img4.c ftab.c ipsw.c normal.c dfu.c recovery.c restore.c asr.c fdr.c limera1n.c download.c locking.c socket.c thread.c jsmn.c json_plist.c
libidevicerestore_la_CFLAGS = $(AM_CFLAGS) -DIDEVICERESTORE_NOMAIN
if WIN32
libidevicerestore_la_CFLAGS += -DCURL_STATICLIB -DZIP_STATIC
endif
libidevicerestore_la_LDFLAGS = $(AM_LDFLAGS)
libidevicerestore_la_LIBADD = $(AM_LDADD)

else
bin_PROGRAMS = idevicerestore

idevicerestore_SOURCES = \
	idevicerestore.c idevicerestore.h \
	endianness.h \
	common.c common.h \
	tss.c tss.h \
	fls.c fls.h \
	mbn.c mbn.h \
	img3.c img3.h \
	img4.c img4.h \
	ftab.c ftab.h \
	ipsw.c ipsw.h \
	normal.c normal.h \
	dfu.c dfu.h \
	recovery.c recovery.h \
	restore.c restore.h \
	asr.c asr.h \
	fdr.c fdr.h \
	limera1n_payload.h \
	limera1n.c limera1n.h \
	download.c download.h \
	locking.c locking.h \
	jsmn.c jsmn.h \
	json_plist.c json_plist.h
if USE_INTERNAL_SHA
idevicerestore_SOURCES += sha1.c sha1.h sha512.c sha512.h fixedint.h
endif
idevicerestore_CFLAGS = $(AM_CFLAGS)
idevicerestore_LDFLAGS = $(AM_LDFLAGS)
idevicerestore_LDADD = $(AM_LDADD)

endif
