#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=openprinting-cups-filters
PKG_VERSION:=1.28.17
PKG_RELEASE:=3

PKG_SOURCE:=cups-filters-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/OpenPrinting/cups-filters/releases/download/1.28.17/
PKG_MD5SUM:=a4f790316c9cc7901162b559325f3d27
PKG_BUILD_DIR:=$(BUILD_DIR)/cups-filters-$(PKG_VERSION)

PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk

# Use TARGET_CXXFLAGS for C++ specific stuff and let the build system handle paths
TARGET_CXXFLAGS += -std=c++17 -DHAVE_CPP_POPPLER_VERSION_H -w
TARGET_CPPFLAGS += -I$(STAGING_DIR)/usr/include/poppler

define Package/openprinting-cups-filters
  SECTION:=Net
  CATEGORY:=Network
  TITLE:=OpenPrinting CUPS filters
  DEPENDS:=+cups +libcupsimage +fontconfig +libijs +libtiff +libjpeg +libpng +poppler +qpdf +libstdcpp +libdbus +libexif +libavahi-client
  URL:=http://www.openprinting.org
  SUBMENU:=Printing
endef

define Package/openprinting-cups-filters/description
	CUPS filters maintained by OpenPrinting.
	The CUPS Filters package contains backends, filters and other software that was once part of the core CUPS distribution but is no longer maintained by Apple Inc.
endef

define Build/Configure
	$(call Build/Configure/Default, \
	--enable-imagefilters \
	--with-pdftops=gs \
	--with-pdftops-path=/usr/bin/gs \
	--with-gs-path=/usr/bin/gs \
	--disable-avahi \
	--disable-glib \
	--disable-gnutls \
	--disable-openssl \
	--disable-cdsassl \
	--disable-ssl \
	--disable-gssapi \
	--with-fontdir=/usr/share/fonts \
	--with-cups-config=$(STAGING_DIR)/usr/bin/cups-config \
	--with-fontconfig-cflags="-I$(STAGING_DIR)/usr/include" \
	--with-fontconfig-libs="-L$(STAGING_DIR)/usr/lib -lfontconfig" \
	--with-poppler-cflags="-I$(STAGING_DIR)/usr/include/poppler/cpp -I$(STAGING_DIR)/usr/include/poppler" \
	--with-poppler-libs="-L$(STAGING_DIR)/usr/lib -lpoppler -lpoppler-cpp" \
	)
endef

# 3. The "Surgery": This runs after Configure and before Compile
define Build/Compile
	# 1. Strip the poisoned host path
	find $(PKG_BUILD_DIR) -name Makefile -exec $(SED) 's|-I/usr/include||g' {} +
	# 2. Safety check: ensure any stray poppler paths have the -I prefix
	find $(PKG_BUILD_DIR) -name Makefile -exec $(SED) 's| /poppler|-I$(STAGING_DIR)/usr/include/poppler|g' {} +
	$(call Build/Compile/Default)
endef

define Package/openprinting-cups-filters/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/usr/lib/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/* $(1)/usr/bin/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/* $(1)/usr/lib/
	$(CP) -r $(PKG_INSTALL_DIR)/usr/lib/cups $(1)/usr/lib/
	$(INSTALL_DIR) $(1)/usr/share/cups $(1)/usr/share/ppd/cupsfilters
	$(CP) -r $(PKG_INSTALL_DIR)/usr/share/cups/* $(1)/usr/share/cups
	$(CP) -r $(PKG_INSTALL_DIR)/usr/share/ppd/cupsfilters/* $(1)/usr/share/ppd/cupsfilters
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/include/* $(1)/usr/include/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/* $(1)/usr/lib
endef

define Package/preinst
#!/bin/sh
# Be on the safe side: get rid of any files from CUPS 1.5.X filters.
# See OpenPrinting's cups-filters-1.0.X/INSTALL
rm /usr/lib/cups/backend/serial
rm /usr/lib/cups/backend/parallel
rm /usr/lib/cups/filter/bannertops
rm /usr/lib/cups/filter/commandtoescpx
rm /usr/lib/cups/filter/commandtopclx
rm /usr/lib/cups/filter/imagetops
rm /usr/lib/cups/filter/imagetoraster
rm /usr/lib/cups/filter/pdftops
rm /usr/lib/cups/filter/rastertoescpx
rm /usr/lib/cups/filter/rastertopclx
rm /usr/lib/cups/filter/texttops
rm /usr/share/cups/banners/*
rm /usr/share/cups/data/testprint
rm /usr/share/cups/data/psglyphs
rm /usr/share/cups/fonts/*
endef

define Package/postinst
#!/bin/sh
/etc/init.d/cupsd stop
/etc/init.d/cupsd start
endef

$(eval $(call BuildPackage,openprinting-cups-filters))

