#
# Copyright (C) 2006-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=openssl
PKG_VERSION:=3.6.0-beta1
PKG_RELEASE:=1
PKG_BUILD_FLAGS:=no-mips16 gc-sections no-lto

PKG_BUILD_PARALLEL:=1

PKG_BASE:=$(subst $(space),.,$(wordlist 1,2,$(subst .,$(space),$(PKG_VERSION))))
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:= \
	https://www.openssl.org/source/ \
	https://www.openssl.org/source/old/$(PKG_BASE)/ \
	https://github.com/openssl/openssl/releases/download/$(PKG_NAME)-$(PKG_VERSION)/

# SHA-256 of openssl-3.6.0-beta1.tar.gz from https://www.openssl.org/source/
# Verify with: sha256sum openssl-3.6.0-beta1.tar.gz
PKG_HASH:=

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE.txt
PKG_MAINTAINER:=Eneas U de Queiroz <cotequeiroz@gmail.com>
PKG_CPE_ID:=cpe:/a:openssl:openssl
PKG_CONFIG_DEPENDS:= \
	CONFIG_OPENSSL_BETA_ENGINE \
	CONFIG_OPENSSL_BETA_ENGINE_BUILTIN \
	CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_AFALG \
	CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO \
	CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK \
	CONFIG_OPENSSL_BETA_NO_DEPRECATED \
	CONFIG_OPENSSL_BETA_OPTIMIZE_SPEED \
	CONFIG_OPENSSL_BETA_PREFER_CHACHA_OVER_GCM \
	CONFIG_OPENSSL_BETA_SMALL_FOOTPRINT \
	CONFIG_OPENSSL_BETA_KTLS \
	CONFIG_OPENSSL_BETA_WITH_ARIA \
	CONFIG_OPENSSL_BETA_WITH_ASM \
	CONFIG_OPENSSL_BETA_WITH_ASYNC \
	CONFIG_OPENSSL_BETA_WITH_BLAKE2 \
	CONFIG_OPENSSL_BETA_WITH_CAMELLIA \
	CONFIG_OPENSSL_BETA_WITH_CHACHA_POLY1305 \
	CONFIG_OPENSSL_BETA_WITH_CMS \
	CONFIG_OPENSSL_BETA_WITH_COMPRESSION \
	CONFIG_OPENSSL_BETA_WITH_DTLS \
	CONFIG_OPENSSL_BETA_WITH_EC2M \
	CONFIG_OPENSSL_BETA_WITH_ERROR_MESSAGES \
	CONFIG_OPENSSL_BETA_WITH_IDEA \
	CONFIG_OPENSSL_BETA_WITH_MDC2 \
	CONFIG_OPENSSL_BETA_WITH_NPN \
	CONFIG_OPENSSL_BETA_WITH_PSK \
	CONFIG_OPENSSL_BETA_WITH_RFC3779 \
	CONFIG_OPENSSL_BETA_WITH_SEED \
	CONFIG_OPENSSL_BETA_WITH_SM234 \
	CONFIG_OPENSSL_BETA_WITH_SRP \
	CONFIG_OPENSSL_BETA_WITH_SSE2 \
	CONFIG_OPENSSL_BETA_WITH_TLS13 \
	CONFIG_OPENSSL_BETA_WITH_WHIRLPOOL

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/openssl-module.mk

ifneq ($(CONFIG_CCACHE),)
HOSTCC=$(HOSTCC_NOCACHE)
HOSTCXX=$(HOSTCXX_NOCACHE)
endif

STABLE_FILES_DIR:=$(TOPDIR)/package/libs/openssl/files

define Package/openssl-beta/Default
  TITLE:=Open source SSL toolkit (beta)
  URL:=https://www.openssl.org/
  SECTION:=libs
  CATEGORY:=Libraries
endef

define Package/libopenssl-beta/config
source "$(SOURCE)/Config.in"
endef

define Package/openssl-beta/Default/description
The OpenSSL Project is a collaborative effort to develop a robust,
commercial-grade, full-featured, and Open Source toolkit implementing the
Transport Layer Security (TLS) protocol as well as a full-strength
general-purpose cryptography library.

This is the BETA variant ($(PKG_VERSION)), intended for testing only.
It installs alongside or in place of the stable libopenssl package, but
cannot be installed at the same time as it.
endef

define Package/libopenssl-beta
$(call Package/openssl-beta/Default)
  SUBMENU:=SSL
  DEPENDS:=+OPENSSL_BETA_WITH_COMPRESSION:zlib \
	   +OPENSSL_BETA_ENGINE_BUILTIN_AFALG:kmod-crypto-user \
	   +OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO:kmod-cryptodev \
	   +OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK:kmod-crypto-hw-padlock \
	   +(arm||armeb||mips||mipsel||powerpc||arc):libatomic
  TITLE+= (libraries, beta)
  ABI_VERSION:=$(firstword $(subst .,$(space),$(PKG_VERSION)))
  PROVIDES:=libopenssl
  CONFLICTS:=libopenssl
  MENU:=1
endef

define Package/libopenssl-beta/description
$(call Package/openssl-beta/Default/description)
This package contains the OpenSSL shared libraries, needed by other programs.
Provides: libopenssl — satisfies dependencies on the stable library package.
Conflicts: libopenssl — only one variant may be installed at a time.
endef

define Package/openssl-beta-util
  $(call Package/openssl-beta/Default)
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libopenssl-beta +libopenssl-beta-conf
  TITLE+= (utility, beta)
  PROVIDES:=openssl-util
  CONFLICTS:=openssl-util
endef

define Package/openssl-beta-util/description
$(call Package/openssl-beta/Default/description)
This package contains the OpenSSL command-line utility (beta variant).
endef

define Package/libopenssl-beta-conf
  $(call Package/openssl-beta/Default)
  SUBMENU:=SSL
  TITLE:=/etc/ssl/openssl.cnf config file (beta)
  DEPENDS:=libopenssl-beta
  PROVIDES:=libopenssl-conf
  CONFLICTS:=libopenssl-conf
endef

define Package/libopenssl-beta-conf/conffiles
/etc/ssl/openssl.cnf
$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO),/etc/ssl/modules.cnf.d/devcrypto.cnf)
$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK),/etc/ssl/modules.cnf.d/padlock.cnf)
endef

define Package/libopenssl-beta-conf/description
$(call Package/openssl-beta/Default/description)
This package installs the OpenSSL configuration file /etc/ssl/openssl.cnf.
endef

ifneq ($(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK)$(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO),)
define Package/libopenssl-beta-conf/postinst
#!/bin/sh

add_engine_config() {
	if [ -z "$${IPKG_INSTROOT}" ] && uci -q get "openssl.$$1" >/dev/null; then
		[ "$$(uci -q get "openssl.$$1.builtin")" = 1 ] && return
		uci set "openssl.$$1.builtin=1" && uci commit openssl
		return
	fi
}

$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO),add_engine_config devcrypto)
$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK),add_engine_config padlock)
endef
endif

$(eval $(call Package/openssl/add-provider,legacy,libopenssl-beta-legacy))
define Package/libopenssl-beta-legacy
  $(call Package/openssl-beta/Default)
  SECTION:=libs
  CATEGORY:=Libraries
  SUBMENU:=SSL
  DEPENDS:=libopenssl-beta +libopenssl-beta-conf
  TITLE:=OpenSSL beta legacy provider
  PROVIDES:=libopenssl-legacy
  CONFLICTS:=libopenssl-legacy
endef

define Package/libopenssl-beta-legacy/description
The OpenSSL legacy provider supplies OpenSSL implementations of algorithms that
have been deemed legacy (beta variant).
endef

$(eval $(call Package/openssl/add-engine,afalg,libopenssl-beta-afalg))
define Package/libopenssl-beta-afalg
  $(call Package/openssl-beta/Default)
  SECTION:=libs
  CATEGORY:=Libraries
  SUBMENU:=SSL
  DEPENDS:=libopenssl-beta +libopenssl-beta-conf @OPENSSL_BETA_ENGINE \
	   @KERNEL_AIO +PACKAGE_libopenssl-beta-afalg:kmod-crypto-user \
	   @!OPENSSL_BETA_ENGINE_BUILTIN
  TITLE:=AFALG hardware acceleration engine (beta)
  PROVIDES:=libopenssl-afalg
  CONFLICTS:=libopenssl-afalg
endef

define Package/libopenssl-beta-afalg/description
This package adds an engine that enables hardware acceleration
through the AF_ALG kernel interface (beta variant).
endef

$(eval $(call Package/openssl/add-engine,devcrypto,libopenssl-beta-devcrypto))
define Package/libopenssl-beta-devcrypto
  $(call Package/openssl-beta/Default)
  SECTION:=libs
  CATEGORY:=Libraries
  SUBMENU:=SSL
  DEPENDS:=libopenssl-beta +libopenssl-beta-conf @OPENSSL_BETA_ENGINE \
	   +PACKAGE_libopenssl-beta-devcrypto:kmod-cryptodev \
	   @!OPENSSL_BETA_ENGINE_BUILTIN
  TITLE:=/dev/crypto hardware acceleration engine (beta)
  PROVIDES:=libopenssl-devcrypto
  CONFLICTS:=libopenssl-devcrypto
endef

define Package/libopenssl-beta-devcrypto/description
This package adds an engine that enables hardware acceleration
through the /dev/crypto kernel interface (beta variant).
endef

$(eval $(call Package/openssl/add-engine,padlock,libopenssl-beta-padlock))
define Package/libopenssl-beta-padlock
  $(call Package/openssl-beta/Default)
  SECTION:=libs
  CATEGORY:=Libraries
  SUBMENU:=SSL
  DEPENDS:=libopenssl-beta +libopenssl-beta-conf @OPENSSL_BETA_ENGINE \
	   @TARGET_x86 +PACKAGE_libopenssl-beta-padlock:kmod-crypto-hw-padlock \
	   @!OPENSSL_BETA_ENGINE_BUILTIN
  TITLE:=VIA Padlock hardware acceleration engine (beta)
  PROVIDES:=libopenssl-padlock
  CONFLICTS:=libopenssl-padlock
endef

define Package/libopenssl-beta-padlock/description
This package adds an engine that enables VIA Padlock hardware acceleration (beta variant).
endef

OPENSSL_OPTIONS:= shared no-tests

ifndef CONFIG_OPENSSL_BETA_WITH_BLAKE2
  OPENSSL_OPTIONS += no-blake2
endif

ifndef CONFIG_OPENSSL_BETA_WITH_CHACHA_POLY1305
  OPENSSL_OPTIONS += no-chacha no-poly1305
else
  ifdef CONFIG_OPENSSL_BETA_PREFER_CHACHA_OVER_GCM
    OPENSSL_OPTIONS += -DOPENSSL_PREFER_CHACHA_OVER_GCM
  endif
endif

ifndef CONFIG_OPENSSL_BETA_WITH_ASYNC
  OPENSSL_OPTIONS += no-async
endif

ifndef CONFIG_OPENSSL_BETA_WITH_EC2M
  OPENSSL_OPTIONS += no-ec2m
endif

ifndef CONFIG_OPENSSL_BETA_WITH_ERROR_MESSAGES
  OPENSSL_OPTIONS += no-err
endif

ifndef CONFIG_OPENSSL_BETA_WITH_TLS13
  OPENSSL_OPTIONS += no-tls1_3
endif

ifndef CONFIG_OPENSSL_BETA_WITH_ARIA
  OPENSSL_OPTIONS += no-aria
endif

ifndef CONFIG_OPENSSL_BETA_WITH_SM234
  OPENSSL_OPTIONS += no-sm2 no-sm3 no-sm4
endif

ifndef CONFIG_OPENSSL_BETA_WITH_CAMELLIA
  OPENSSL_OPTIONS += no-camellia
endif

ifndef CONFIG_OPENSSL_BETA_WITH_IDEA
  OPENSSL_OPTIONS += no-idea
endif

ifndef CONFIG_OPENSSL_BETA_WITH_SEED
  OPENSSL_OPTIONS += no-seed
endif

ifndef CONFIG_OPENSSL_BETA_WITH_MDC2
  OPENSSL_OPTIONS += no-mdc2
endif

ifndef CONFIG_OPENSSL_BETA_WITH_WHIRLPOOL
  OPENSSL_OPTIONS += no-whirlpool
endif

ifndef CONFIG_OPENSSL_BETA_WITH_CMS
  OPENSSL_OPTIONS += no-cms
endif

ifndef CONFIG_OPENSSL_BETA_WITH_RFC3779
  OPENSSL_OPTIONS += no-rfc3779
endif

ifdef CONFIG_OPENSSL_BETA_NO_DEPRECATED
  OPENSSL_OPTIONS += no-deprecated
endif

ifeq ($(CONFIG_OPENSSL_BETA_OPTIMIZE_SPEED),y)
  TARGET_CFLAGS := $(filter-out -O%,$(TARGET_CFLAGS)) -O3
endif

ifeq ($(CONFIG_OPENSSL_BETA_SMALL_FOOTPRINT),y)
  OPENSSL_OPTIONS += -DOPENSSL_SMALL_FOOTPRINT
endif

ifdef CONFIG_OPENSSL_BETA_KTLS
  OPENSSL_OPTIONS += enable-ktls
endif

ifdef CONFIG_OPENSSL_BETA_ENGINE
  ifdef CONFIG_OPENSSL_BETA_ENGINE_BUILTIN
    OPENSSL_OPTIONS += disable-dynamic-engine
    ifndef CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_AFALG
      OPENSSL_OPTIONS += no-afalgeng
    endif
    ifdef CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO
      OPENSSL_OPTIONS += enable-devcryptoeng
    endif
    ifndef CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK
      OPENSSL_OPTIONS += no-padlockeng
    endif
  else
    ifdef CONFIG_PACKAGE_libopenssl-beta-devcrypto
      OPENSSL_OPTIONS += enable-devcryptoeng
    endif
    ifndef CONFIG_PACKAGE_libopenssl-beta-afalg
      OPENSSL_OPTIONS += no-afalgeng
    endif
    ifndef CONFIG_PACKAGE_libopenssl-beta-padlock
      OPENSSL_OPTIONS += no-padlockeng
    endif
  endif
else
  OPENSSL_OPTIONS += no-engine
endif

ifndef CONFIG_OPENSSL_BETA_WITH_DTLS
  OPENSSL_OPTIONS += no-dtls
endif

ifdef CONFIG_OPENSSL_BETA_WITH_COMPRESSION
  OPENSSL_OPTIONS += zlib-dynamic
else
  OPENSSL_OPTIONS += no-comp
endif

ifndef CONFIG_OPENSSL_BETA_WITH_NPN
  OPENSSL_OPTIONS += no-nextprotoneg
endif

ifndef CONFIG_OPENSSL_BETA_WITH_PSK
  OPENSSL_OPTIONS += no-psk
endif

ifndef CONFIG_OPENSSL_BETA_WITH_SRP
  OPENSSL_OPTIONS += no-srp
endif

ifndef CONFIG_OPENSSL_BETA_WITH_ASM
  OPENSSL_OPTIONS += no-asm
endif

ifdef CONFIG_i386
  ifndef CONFIG_OPENSSL_BETA_WITH_SSE2
    OPENSSL_OPTIONS += no-sse2
  endif
endif

OPENSSL_TARGET:=linux-$(call qstrip,$(CONFIG_ARCH))-openwrt

STAMP_CONFIGURED := $(STAMP_CONFIGURED)_$(shell echo $(OPENSSL_OPTIONS) | $(MKHASH) md5)

TARGET_CFLAGS += $(FPIC)

define Build/Configure
	(cd $(PKG_BUILD_DIR); \
		CFLAGS="$(TARGET_CFLAGS)" \
		CPPFLAGS="$(TARGET_CPPFLAGS)" \
		LDFLAGS="$(TARGET_LDFLAGS)" \
		./Configure $(OPENSSL_TARGET) \
			--prefix=/usr \
			--libdir=lib \
			--openssldir=/etc/ssl \
			--cross-compile-prefix="$(TARGET_CROSS)" \
			$(OPENSSL_OPTIONS) && \
		{ [ -f $(STAMP_CONFIGURED) ] || make clean; } \
	)
endef

define Build/Compile
	+$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
		CC="$(TARGET_CC)" \
		SOURCE_DATE_EPOCH=$(SOURCE_DATE_EPOCH) \
		all
	$(MAKE) -C $(PKG_BUILD_DIR) \
		CC="$(TARGET_CC)" \
		DESTDIR="$(PKG_INSTALL_DIR)" \
		install_sw install_ssldirs
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/openssl $(1)/usr/include/
	$(INSTALL_DIR) $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/lib{crypto,ssl}.{a,so*} $(1)/usr/lib/
	$(INSTALL_DIR) $(1)/usr/lib/pkgconfig
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/pkgconfig/{openssl,libcrypto,libssl}.pc $(1)/usr/lib/pkgconfig/
	[ -n "$(TARGET_LDFLAGS)" ] && $(SED) 's#$(TARGET_LDFLAGS)##g' $(1)/usr/lib/pkgconfig/{openssl,libcrypto,libssl}.pc || true
endef

define Package/libopenssl-beta/install
	$(INSTALL_DIR) $(1)/etc/ssl/certs
	$(INSTALL_DIR) $(1)/etc/ssl/private
	chmod 0700 $(1)/etc/ssl/private
	$(INSTALL_DIR) $(1)/usr/lib
	$(INSTALL_DATA) $(PKG_INSTALL_DIR)/usr/lib/libcrypto.so.* $(1)/usr/lib/
	$(INSTALL_DATA) $(PKG_INSTALL_DIR)/usr/lib/libssl.so.* $(1)/usr/lib/
	$(if $(CONFIG_OPENSSL_BETA_ENGINE),$(INSTALL_DIR) $(1)/usr/lib/$(ENGINES_DIR))
endef

define Package/libopenssl-beta-conf/install
	$(INSTALL_DIR) $(1)/etc/ssl/modules.cnf.d $(1)/etc/config $(1)/etc/init.d
	$(CP) $(PKG_INSTALL_DIR)/etc/ssl/openssl.cnf $(1)/etc/ssl/
	$(INSTALL_BIN) $(STABLE_FILES_DIR)/openssl.init $(1)/etc/init.d/openssl
	$(SED) 's!%ENGINES_DIR%!/usr/lib/$(ENGINES_DIR)!' $(1)/etc/init.d/openssl
	touch $(1)/etc/config/openssl
	$(if $(CONFIG_OPENSSL_BETA_ENGINE),,
		$(SED) 's!engines = engines_sect!#&!' $(1)/etc/ssl/openssl.cnf)
	$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_DEVCRYPTO),
		$(CP) $(STABLE_FILES_DIR)/devcrypto.cnf $(1)/etc/ssl/modules.cnf.d/
		echo -e "config engine 'devcrypto'\n\toption enabled '1'\n\toption builtin '1'" >> $(1)/etc/config/openssl)
	$(if $(CONFIG_OPENSSL_BETA_ENGINE_BUILTIN_PADLOCK),
		$(CP) $(STABLE_FILES_DIR)/padlock.cnf $(1)/etc/ssl/modules.cnf.d/
		echo -e "\nconfig engine 'padlock'\n\toption enabled '1'\n\toption builtin '1'" >> $(1)/etc/config/openssl)
endef

define Package/openssl-beta-util/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/openssl $(1)/usr/bin/
endef

$(eval $(call BuildPackage,libopenssl-beta))
$(eval $(call BuildPackage,libopenssl-beta-conf))
$(eval $(call BuildPackage,libopenssl-beta-afalg))
$(eval $(call BuildPackage,libopenssl-beta-devcrypto))
$(eval $(call BuildPackage,libopenssl-beta-legacy))
$(eval $(call BuildPackage,libopenssl-beta-padlock))
$(eval $(call BuildPackage,openssl-beta-util))
