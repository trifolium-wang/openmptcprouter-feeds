#
# Based on package from https://github.com/openwrt-develop/ndpi-netfilter/
# Copyright (C) 2018-2019 Ycarus (Yannick Chabanois) <ycarus@zugaina.org>
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=ndpi-netfilter2
PKG_RELEASE:=2
PKG_REV:=77d882638232fb9236123b50b9aac8bccc409ed9
PKG_VERSION:=3.2-$(PKG_REV)

PKG_SOURCE_PROTO:=git
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_URL:=https://github.com/vel21ripn/nDPI.git
PKG_SOURCE_VERSION:=$(PKG_REV)

PKG_BUILD_DIR:=$(KERNEL_BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/iptables-mod-ndpi
	SUBMENU:=Firewall
	SECTION:=net
	CATEGORY:=Network
	TITLE:=ndpi successor of OpenDPI
	URL:=http://www.ntop.org/products/ndpi/
#	DEPENDS:=+iptables +iptables-mod-conntrack-extra +kmod-ipt-ndpi +libpcap
	DEPENDS:=+iptables +kmod-ipt-ndpi +libpcap
	MAINTAINER:=Ycarus (Yannick Chabanois) <ycarus@zugaina.org>
endef

define Package/iptables-mod-ndpi/description
  nDPI is a ntop-maintained superset of the popular OpenDPI library
endef

#CONFIGURE_CMD = ./autogen.sh
#CONFIGURE_ARGS += --with-pic
MAKE_PATH := ndpi-netfilter

MAKE_FLAGS += \
    KERNEL_DIR="$(LINUX_DIR)" \
    MODULES_DIR="$(TARGET_MODULES_DIR)" \
    ARCH="$(LINUX_KARCH)" \
    CROSS_COMPILE="$(TARGET_CROSS)" \
    NDPI_PATH=$(PKG_BUILD_DIR)/ndpi-netfilter

define Build/Configure
	cd $(PKG_BUILD_DIR) && \
	    CC="" ./autogen.sh \
	    --host="$(GNU_TARGET_NAME)"
#	    --with-pic \
#	    CFLAGS="$(TARGET_CFLAGS)" \
#	    HOST_CFLAGS="$(BUILD_CFLAGS)" \
#	    BUILD_CPPFLAGS="$(TARGET_CPPLAGS)" \
#	    BUILD_CFLAGS="$(TARGET_CFLAGS)" \
#	    BUILD_LDFLAGS="$(TARGET_LDFLAGS)"
endef

define Build/Compile
	(cd $(PKG_BUILD_DIR)/src/lib &&\
	    gcc -g -O2 -fPIC -DPIC -DNDPI_LIB_COMPILATION -I../../src/include/ -I../../src/lib/third_party/include/ ndpi_network_list_compile.c -o ndpi_network_list_compile &&\
	    ./ndpi_network_list_compile -o ndpi_network_list.c.inc ndpi_network_list_std.yaml ndpi_network_list_tor.yaml)
	$(MAKE) $(MAKE_FLAGS) -C $(PKG_BUILD_DIR)/ndpi-netfilter
endef

define Package/iptables-mod-ndpi/install
	$(INSTALL_DIR) $(1)/usr/lib/iptables
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ndpi-netfilter/ipt/libxt_ndpi.so $(1)/usr/lib/iptables
endef

define KernelPackage/ipt-ndpi
	SUBMENU:=Netfilter Extensions
	TITLE:= nDPI net netfilter module
	DEPENDS:=+kmod-nf-conntrack +kmod-nf-conntrack-netlink +kmod-ipt-compat-xtables +libpcap
	KCONFIG:=CONFIG_NF_CONNTRACK_LABELS=y \
	    CONFIG_NETFILTER_XT_MATCH_CONNLABEL=y
	FILES:= $(PKG_BUILD_DIR)/ndpi-netfilter/src/xt_ndpi.ko
	AUTOLOAD:=$(call AutoProbe,xt_ndpi)
endef

$(eval $(call BuildPackage,iptables-mod-ndpi))
$(eval $(call KernelPackage,ipt-ndpi))
