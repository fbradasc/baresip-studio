MODULES := openssl opus ogg libzrtp rem re baresip

PATCHES     := re_types.h

UPDATES     := $(patsubst %,%_update,$(MODULES))
TOCLEAN     := $(patsubst %,%_clean,$(MODULES))
TOWIPE      := $(patsubst %,%_wipe,$(MODULES))
TOWIPEALL   := $(patsubst %,%_wipeall,$(MODULES))
TOBUILD     := $(patsubst %,%_build,$(MODULES))
TOUNINSTALL := $(patsubst %,%_uninstall,$(MODULES))
TOPATCH     := $(patsubst %,%_patch,$(PATCHES))

# Paths to your Android SDK/NDK
# currently it compiles only with android-ndk-r15c
ANDROID_NDK := /opt/android-ndk
ANDROID_SDK := /opt/android-sdk

#
# Android API-levels:
#
# API-Level:      Version:
# ---------       -------
#     17           4.2.x
#     18           4.3.x
#     19           4.4 - 4.4.4
#     21           5.0
#     22           6.0
#     23           7.0
#
ANDROID_API := 23
PLATFORM    := android-$(ANDROID_API)


TRIPLE      := arm-linux-androideabi


# Path to install binaries on your Android-device
TARGET_PATH=/data/local/tmp

# Config path where .baresip directory is located
CONFIG_PATH=/data/local/tmp


OS          := $(shell uname -s | tr "[A-Z]" "[a-z]")

ifeq ($(OS),linux)
	HOST_OS   := linux-x86_64
endif
ifeq ($(OS),darwin)
	HOST_OS   := darwin-x86_64
endif


# Tools
SYSROOT     := $(ANDROID_NDK)/platforms/$(PLATFORM)/arch-arm
SYSROOT_INC := $(ANDROID_NDK)/sysroot

PREBUILT    := $(ANDROID_NDK)/toolchains/arm-linux-androideabi-4.9/prebuilt
BIN         := $(PREBUILT)/$(HOST_OS)/bin
CC          := $(BIN)/arm-linux-androideabi-gcc
CXX         := $(BIN)/arm-linux-androideabi-g++
RANLIB      := $(BIN)/arm-linux-androideabi-ranlib
AR          := $(BIN)/arm-linux-androideabi-ar
ADB         := $(ANDROID_SDK)/platform-tools/adb
PWD         := $(shell pwd)

# Compiler and Linker Flags
#
# NOTE: use -isystem to avoid warnings in system header files
CFLAGS    := \
	-D__ANDROID_API__=$(ANDROID_API) \
	-isystem $(SYSROOT_INC)/usr/include/ \
	-isystem $(SYSROOT_INC)/usr/include/$(TRIPLE) \
	-I$(PWD)/openssl/include \
	-I$(PWD)/opus/include_opus \
	-I$(PWD)/libzrtp/include \
	-I$(PWD)/libzrtp/third_party/bnlib \
	-I$(PWD)/libzrtp/third_party/bgaes \
	-march=armv7-a \
	-fPIE -fPIC \
	-DCONFIG_PATH='\"$(CONFIG_PATH)\"'
LFLAGS    := -L$(SYSROOT)/usr/lib/ \
	-L$(PWD)/openssl \
	-L$(PWD)/opus/.libs \
	-L$(PWD)/libzrtp \
	-L$(PWD)/libzrtp/third_party/bnlib \
	-fPIE -pie
LFLAGS    += --sysroot=$(ANDROID_NDK)/platforms/$(PLATFORM)/arch-arm


COMMON_FLAGS := CC=$(CC) \
		CXX=$(CXX) \
		RANLIB=$(RANLIB) \
		AR=$(AR) \
		EXTRA_CFLAGS="$(CFLAGS) -DANDROID" \
		EXTRA_CXXFLAGS="$(CFLAGS) -DANDROID" \
		EXTRA_LFLAGS="$(LFLAGS)" \
		SYSROOT=$(SYSROOT_INC)/usr \
		SYSROOT_ALT=$(SYSROOT)/usr \
		SHARE_PATH="$(TARGET_PATH)/share" \
		HAVE_LIBRESOLV= \
		HAVE_RESOLV= \
		HAVE_PTHREAD=1 \
		HAVE_PTHREAD_RWLOCK=1 \
		HAVE_LIBPTHREAD= \
		HAVE_INET_PTON=1 \
		HAVE_INET6=1 \
		HAVE_GETIFADDRS= \
		PEDANTIC= \
		OS=linux ARCH=arm \
		USE_OPENSSL=yes \
		USE_OPENSSL_DTLS=yes \
		USE_OPENSSL_SRTP=yes \
		ANDROID=yes

EXTRA_MODULES := g711 stdio opensles dtls_srtp echo aubridge

ifneq ("$(wildcard $(PWD)/opus)","")
	EXTRA_MODULES := $(EXTRA_MODULES) opus
endif

ifneq ("$(wildcard $(PWD)/libzrtp)","")
	EXTRA_MODULES := $(EXTRA_MODULES) zrtp
endif

build: prepare $(TOBUILD)

.PHONY: rem_build re_build
rem_build re_build: Makefile
	@rm -f $(@:_build=)/lib$(@:_build=).*
	@make lib$(@:_build=).a -C $(@:_build=) $(COMMON_FLAGS)

libbaresip.a_build:	Makefile
	@rm -f baresip/$(@:_build=) baresip/src/static.c
	PKG_CONFIG_LIBDIR="$(SYSROOT)/usr/lib/pkgconfig" \
	make $(@:_build=) -C baresip $(COMMON_FLAGS) STATIC=1 \
		LIBRE_SO=$(PWD)/re LIBREM_PATH=$(PWD)/rem \
	        MOD_AUTODETECT= \
		EXTRA_MODULES="$(EXTRA_MODULES)"

.PHONY: baresip_build
baresip_build: libbaresip.a_build Makefile
	@rm -f baresip/$(@:_build=) baresip/src/static.c
	PKG_CONFIG_LIBDIR="$(SYSROOT)/usr/lib/pkgconfig" \
	make $(@:_build=) -C baresip $(COMMON_FLAGS) STATIC=1 \
		LIBRE_SO=$(PWD)/re LIBREM_PATH=$(PWD)/rem \
	        MOD_AUTODETECT= \
		EXTRA_MODULES="$(EXTRA_MODULES)"

install:	baresip_build
	$(ADB) push baresip/baresip $(TARGET_PATH)/baresip

config:
	$(ADB) push .baresip $(TARGET_PATH)/
	$(ADB) push baresip/share $(TARGET_PATH)/share

OPENSSL_FLAGS := \
	threads \
	-D__ANDROID_API__=$(ANDROID_API) \
	-isystem$(SYSROOT_INC)/usr/include \
	-isystem$(SYSROOT_INC)/usr/include/$(TRIPLE) \
	-fPIE -fPIC -pie \
	\
	no-async \
	no-bf \
	no-blake2 \
	no-camellia \
	no-capieng \
	no-cast \
	no-comp \
	no-dso \
	no-engine \
	no-gost \
	no-heartbeats \
	no-idea \
	no-md2 \
	no-md4 \
	no-mdc2 \
	no-psk \
	no-rc2 \
	no-rc4 \
	no-rc5 \
	no-sctp \
	no-seed \
	no-shared \
	no-srp \
	no-ssl3


.PHONY: openssl_build
openssl_build:
	cd openssl && \
		PATH=$(BIN):$(PATH) \
		./Configure android-arm $(OPENSSL_FLAGS) && \
		PATH=$(BIN):$(PATH) \
		ANDROID_DEV=$(SYSROOT)/usr \
		CROSS_SYSROOT="$(SYSROOT)" \
		make build_libs

.PHONY: opus_build
opus_build:
	cd $(@:_build=) && \
		./autogen.sh && \
		rm -rf include_opus && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		./configure --host=arm-linux-androideabi --disable-shared \
			--disable-doc \
			--disable-extra-programs \
			CFLAGS="$(CFLAGS)" && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		make && \
		mkdir include_opus && \
		mkdir include_opus/opus && \
		cp include/* include_opus/opus

.PHONY: ogg_build
ogg_build:
	cd $(@:_build=) && \
		./autogen.sh && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		./configure --host=arm-linux-androideabi --disable-shared CFLAGS="$(CFLAGS)" && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		make

.PHONY: libzrtp_build
libzrtp_build:
	cd $(@:_build=) && \
		./bootstrap.sh && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		./configure --host=arm-linux-androideabi CFLAGS="$(CFLAGS)" && \
		cd third_party/bnlib/ && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		./configure --host=arm-linux-androideabi CFLAGS="$(CFLAGS)" && \
		cd ../.. && \
		CC="$(CC) --sysroot $(SYSROOT)" \
		RANLIB=$(RANLIB) AR=$(AR) PATH=$(BIN):$(PATH) \
		make

emulator:
	@$(ANDROID_SDK)/tools/emulator -avd test

shell:
	@$(ADB) shell

info:
	make $@ -C re $(COMMON_FLAGS)

dump:
	@echo "ANDROID_NDK = $(ANDROID_NDK)"
	@echo "ANDROID_SDK = $(ANDROID_SDK)"
	@echo "HOST_OS     = $(HOST_OS)"
	@make info -C re $(COMMON_FLAGS)

#
# additional targets for `retest'
#

.PHONY: toolchain
toolchain:
	$(ANDROID_NDK)/build/tools/make_standalone_toolchain.py \
		--arch arm \
		--api $(ANDROID_API) \
		--install-dir $(PWD)/toolchain

prepare: $(MODULES) $(UPDATES) patch

clean: $(TOCLEAN)

distclean: clean $(TOWIPE)

wipeall: $(TOWIPEALL)

baresip:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/alfredh/baresip.git

libsrtp:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/cisco/libsrtp.git

libzrtp:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/juha-h/libzrtp

re rem retest:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/creytiv/$@.git

openssl:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/openssl/$@.git

opus ogg:
	@echo
	@echo "Fetching $@"
	@echo
	git clone https://github.com/xiph/$@.git

$(UPDATES):
	@echo
	@echo "Upgrading $@"
	@echo
	cd $(@:_update=) && git pull --all

$(TOWIPE):
	@echo
	@echo "Distcleaning $@"
	@echo
	cd $(@:_wipe=) && make distclean

$(TOWIPEALL):
	@echo
	@echo "Wiping all $@"
	@echo
	rm -rf $(@:_wipeall=)

$(TOCLEAN):
	@echo
	@echo "Cleaning $@"
	@echo
	cd $(@:_clean=) && make clean

patch:
	-for patch in patches/*; \
	do \
		patch -p1 -N < $$patch ; \
	done
