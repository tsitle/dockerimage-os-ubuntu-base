FROM scratch

ARG CF_UBUNTU_RELEASE
ARG CF_UBUNTU_VERSION
ARG CF_CPUARCH_DEB_DIST
ENV CF_CPUARCH_DEB_DIST=${CF_CPUARCH_DEB_DIST}
ARG CF_CPUARCH_DEB_ROOTFS

# copy Ubuntu root-fs
ADD cache/ubuntu_${CF_UBUNTU_RELEASE}/ubuntu-${CF_UBUNTU_RELEASE}-${CF_UBUNTU_VERSION}-core-cloudimg-${CF_CPUARCH_DEB_ROOTFS}-root.tgz /

# set environment variables
ENV DEBIAN_FRONTEND="noninteractive"
ENV HOME="/root" \
LANGUAGE="en_US.UTF-8" \
LANG="en_US.UTF-8" \
TERM="xterm"

# ------------------------------------------------------------------------------------------
# taken from:
#   https://github.com/docker-library/repo-info/blob/master/repos/ubuntu/remote/bionic.md
#
RUN [ -z "$(apt-get indextargets)" ]
RUN \
	set -xe \
	&& echo '#!/bin/sh' > /usr/sbin/policy-rc.d \
	&& echo 'exit 101' >> /usr/sbin/policy-rc.d \
	&& chmod +x /usr/sbin/policy-rc.d \
	&& dpkg-divert --local --rename --add /sbin/initctl \
	&& cp -a /usr/sbin/policy-rc.d /sbin/initctl \
	&& sed -i 's/^exit.*/exit 0/' /sbin/initctl \
	&& echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \
	&& echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean \
	&& echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean \
	&& echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean \
	&& echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages \
	&& echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes \
	&& echo 'Apt::AutoRemove::SuggestsImportant "false";' > /etc/apt/apt.conf.d/docker-autoremove-suggests
RUN \
	mkdir -p /run/systemd \
	&& echo 'docker' > /run/systemd/container
CMD ["/bin/bash"]
# ------------------------------------------------------------------------------------------

RUN \
	apt-get update \
	&& apt-get upgrade -y \
	&& apt-get dist-upgrade -y \
	&& apt-get install -y --no-install-recommends \
			apt-utils \
			locales \
			curl \
			tzdata \
			nano \
			less \
			wget \
			ca-certificates \
	|| exit 1; \
	#
	echo "**** generate locale ****"; \
	locale-gen en_US.UTF-8 \
	&& sed -i \
			-e 's/# de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/' \
			-e 's/# de_DE ISO-8859-1/de_DE ISO-8859-1/' \
			-e 's/# de_DE@euro ISO-8859-15/de_DE@euro ISO-8859-15/' \
			-e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' \
			-e 's/# en_US ISO-8859-1/en_US ISO-8859-1/' \
			-e 's/# en_US@euro ISO-8859-15/en_US@euro ISO-8859-15/' \
			/etc/locale.gen \
	&& dpkg-reconfigure locales \
	|| exit 1; \
	#
	apt-get clean \
	&& rm -rf \
			/tmp/* \
			/var/lib/apt/lists/* \
			/var/tmp/*

# copy bashrc
COPY --chown=0:0 files/bash/dot_bashrc /root/.bashrc

#
ENV DEBIAN_FRONTEND=dialog

WORKDIR /root/
