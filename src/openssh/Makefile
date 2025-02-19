.ONESHELL:
SHELL = /bin/bash
.SHELLFLAGS += -e

MAIN_TARGET = openssh-server_$(OPENSSH_VERSION)_$(CONFIGURED_ARCH).deb
DERIVED_TARGETS = openssh-server-dbgsym_$(OPENSSH_VERSION)_$(CONFIGURED_ARCH).deb

$(addprefix $(DEST)/, $(MAIN_TARGET)): $(DEST)/% :
	# Obtain openssh: https://salsa.debian.org/ssh-team/openssh/-/tree/debian/1%257.9p1-10+deb10u2
	rm -rf ./openssh-server
	git clone https://salsa.debian.org/ssh-team/openssh.git openssh-server
	pushd ./openssh-server

	# Check out tag: debian/1%7.9p1-10+deb10u2
	git checkout -b openssh-src -f 6d9ca74c48d9911342c6ca5aaac8a25974fa2619

	# Apply patch series
	stg init
	stg import -s ../patch/series

	# Build package
ifeq ($(CROSS_BUILD_ENVIRON), y)
	sed -i "s#\-\-with-kerberos5=\/usr#--with-kerberos5-libdir=$$CROSS_PKGS_LIB_PATH#g" debian/rules
	CONFIG_SITE=/etc/dpkg-cross/cross-config.$(CONFIGURED_ARCH) LD_LIBRARY_PATH=$(CROSS_LIB_PATH) dpkg-buildpackage -rfakeroot -b -us -uc -a$(CONFIGURED_ARCH) -Pcross,nocheck -j$(SONIC_CONFIG_MAKE_JOBS) --admindir $(SONIC_DPKG_ADMINDIR)
else
	sudo http_proxy=$(http_proxy) apt-get -y build-dep openssh
	dpkg-buildpackage -rfakeroot -b -us -uc -j$(SONIC_CONFIG_MAKE_JOBS) --admindir $(SONIC_DPKG_ADMINDIR)
endif
	popd

	mv $(DERIVED_TARGETS) $* $(DEST)/

$(addprefix $(DEST)/, $(DERIVED_TARGETS)): $(DEST)/% : $(DEST)/$(MAIN_TARGET)
