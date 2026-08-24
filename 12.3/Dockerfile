# vim:set ft=dockerfile:
FROM ghcr.io/loong64/debian:trixie AS builder

ARG GPG_KEYS=177F4010FE56CA3336300305F1656F24C74CD1D8

RUN set -eux; \
	apt-get update; \
	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
		ca-certificates \
		curl \
		dpkg-dev \
		gcc \
		gpg \
		gpgv \
		libjemalloc2 \
		libtcmalloc-minimal4t64 \
		pwgen \
		tzdata \
		xz-utils \
		zstd \
		dirmngr \
		gpg-agent \
		wget; \
	rm -rf /var/lib/apt/lists/*; \
	GNUPGHOME="$(mktemp -d)"; \
	export GNUPGHOME; \
	gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
	for key in $GPG_KEYS; do \
		gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key"; \
	done; \
	gpg --batch --export "$GPG_KEYS" > /etc/apt/trusted.gpg.d/mariadb.gpg; \
	if command -v gpgconf >/dev/null; then \
		gpgconf --kill all; \
	fi

ARG RUNNER_ARCH
RUN set -eux; \
 	cd /opt; \
 	curl -LO https://github.com/loong64/static-clang-build/raw/refs/heads/main/install-static-clang.sh; \
 	bash ./install-static-clang.sh; \
 	rm -f ./install-static-clang.sh

ENV PATH="/opt/clang/bin:$PATH"
ENV LANG C.UTF-8

# bashbrew-architectures: amd64 arm64v8 ppc64le s390x
ARG MARIADB_VERSION=1:12.3.3+maria~deb13
ENV MARIADB_VERSION $MARIADB_VERSION
# release-status:Stable
# release-support-type:Long Term Support
# (https://downloads.mariadb.org/rest-api/mariadb/)

RUN set -ex; \
	export PYTHONDONTWRITEBYTECODE=1; \
	\
	aptRepo="[ signed-by=/etc/apt/trusted.gpg.d/mariadb.gpg ] http://archive.mariadb.org/mariadb-12.3.3/repo/debian/ trixie main"; \
	echo "deb-src $aptRepo" > /etc/apt/sources.list.d/mariadb.list; \
	apt-get update; \
	tempDir="$(mktemp -d)"; \
	cd "$tempDir"; \
	export DEB_BUILD_OPTIONS="nocheck parallel=$(nproc)"; \
	apt-get build-dep -y "mariadb-server=$MARIADB_VERSION"; \
	apt-get source --compile "mariadb-server=$MARIADB_VERSION"; \
	find . -maxdepth 1 -mindepth 1 -type d -exec rm -rf {} +; \
	cd /; \
	mv "$tempDir" /dist

FROM scratch
COPY --from=builder /dist /dist