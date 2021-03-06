#
# Copyright (C) 2006-2012 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=broadcom-wl
PKG_VERSION:=5.10.56.27.3
PKG_RELEASE:=6

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)_$(ARCH).tar.bz2
PKG_SOURCE_URL:=http://downloads.openwrt.org/sources

PKG_MD5SUM.mipsel:=3363e3a6b3d9d73c49dea870c7834eac
PKG_MD5SUM.mips:=f8de63debc75333d6b4e28193eb051ff
PKG_MD5SUM:=$(PKG_MD5SUM.$(ARCH))

include $(INCLUDE_DIR)/package.mk

define Package/broadcom-wl/Default
  SECTION:=kernel
  CATEGORY:=Kernel modules
  DEPENDS:=@PACKAGE_kmod-brcm-wl||PACKAGE_kmod-brcm-wl-mini
  SUBMENU:=Proprietary BCM43xx WiFi driver
  SUBMENUDEP:=@TARGET_brcm47xx||TARGET_brcm63xx
endef

define KernelPackage/brcm-wl/Default
  $(call Package/broadcom-wl/Default)
  SECTION:=kernel
  DEPENDS:=@TARGET_brcm47xx||TARGET_brcm63xx +wireless-tools
  TITLE:=Kernel driver for BCM43xx chipsets
  FILES:=$(PKG_BUILD_DIR)/driver$(1)/wl.ko $(PKG_BUILD_DIR)/glue/wl_glue.ko
  AUTOLOAD:=$(call AutoLoad,30,wl_glue wl)
endef

define KernelPackage/brcm-wl/Default/description
 This package contains the proprietary wireless driver for the Broadcom 
 BCM43xx chipset.
endef

define KernelPackage/brcm-wl
$(call KernelPackage/brcm-wl/Default,)
  TITLE+= (normal version)
endef

define KernelPackage/brcm-wl/description
$(call KernelPackage/brcm-wl/Default/description)
endef

define KernelPackage/brcm-wl-mini
$(call KernelPackage/brcm-wl/Default,-mini)
  TITLE+= (Legacy version)
endef

define KernelPackage/brcm-wl-mini/description
$(call KernelPackage/brcm-wl/Default/description)
endef

define Package/wlc
$(call Package/broadcom-wl/Default)
  TITLE:=wl driver setup utility
endef

define Package/wlc/description
 This package contains an utility for initializing the proprietary Broadcom 
 wl driver.
endef

define Package/wl
$(call Package/broadcom-wl/Default)
  TITLE:=Proprietary Broadcom wl driver config utility
endef

define Package/wl/description
 This package contains the proprietary utility (wl) for configuring the 
 proprietary Broadcom wl driver.
endef

define Package/nas
$(call Package/broadcom-wl/Default)
  TITLE:=Proprietary Broadcom WPA/WPA2 authenticator
endef

define Package/nas/description
 This package contains the proprietary WPA/WPA2 authenticator (nas) for the 
 proprietary Broadcom wl driver.
endef

MAKE_KMOD := $(MAKE) -C "$(LINUX_DIR)" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		PATH="$(TARGET_PATH)" \
		SUBDIRS="$(PKG_BUILD_DIR)/kmod" \

define Build/Prepare
	$(call Build/Prepare/Default)
	$(CP) $(PKG_BUILD_DIR)/driver $(PKG_BUILD_DIR)/driver-mini
	$(CP) ./src/glue $(PKG_BUILD_DIR)/glue
endef

define Build/Compile
	# Compile the kernel part
	$(MAKE_KMOD) \
		SUBDIRS="$(PKG_BUILD_DIR)/driver" \
		MODFLAGS="-DMODULE -mlong-calls" \
		modules

	$(MAKE_KMOD) \
		SUBDIRS="$(PKG_BUILD_DIR)/driver-mini" \
		MODFLAGS="-DMODULE -mlong-calls" \
		BUILD_TYPE="wl_apsta_mini" \
		modules

	# Compile glue driver
	$(MAKE_KMOD) -C "$(LINUX_DIR)" \
		SUBDIRS="$(PKG_BUILD_DIR)/glue" \
		modules                                        

	# Compile libshared
	$(MAKE) -C $(PKG_BUILD_DIR)/shared \
		$(TARGET_CONFIGURE_OPTS) \
		CFLAGS="$(TARGET_CFLAGS) -I. -I$(PKG_BUILD_DIR)/driver/include" \
		all

	$(TARGET_CC) -o $(PKG_BUILD_DIR)/wlc \
		-I$(PKG_BUILD_DIR)/shared -I$(PKG_BUILD_DIR)/driver/include \
		./src/wlc.c $(PKG_BUILD_DIR)/shared/libshared.a

	$(TARGET_CC) -o $(PKG_BUILD_DIR)/nas \
		$(PKG_BUILD_DIR)/nas_exe.o \
		$(PKG_BUILD_DIR)/shared/libshared.a

	$(TARGET_CC) -o $(PKG_BUILD_DIR)/wl \
		$(PKG_BUILD_DIR)/wl_exe.o \
		$(PKG_BUILD_DIR)/shared/libshared.a
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_BUILD_DIR)/shared/libshared.a $(1)/usr/lib/
endef

define Package/wlc/install
	$(CP) ./files/* $(1)/
	$(INSTALL_DIR) $(1)/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/wlc $(1)/sbin/
endef

define Package/wlc/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || /etc/init.d/wlunbind enable || true
endef

define Package/wl/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/wl $(1)/usr/sbin/
endef

define Package/nas/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/nas $(1)/usr/sbin/
	ln -sf nas $(1)/usr/sbin/nas4not
	ln -sf nas $(1)/usr/sbin/nas4wds
endef

$(eval $(call KernelPackage,brcm-wl))
$(eval $(call KernelPackage,brcm-wl-mini))
$(eval $(call BuildPackage,wlc))
$(eval $(call BuildPackage,wl))
$(eval $(call BuildPackage,nas))
