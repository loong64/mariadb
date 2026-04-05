FROM ghcr.io/loong64/debian:trixie-slim AS builder

ARG GPG_KEYS=177F4010FE56CA3336300305F1656F24C74CD1D8

RUN set -eux; \
	apt-get update; \
	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
		ca-certificates \
        dpkg-dev \
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
	gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys F806E8C4741675050854E1E33617A873C05597AD; \
	for key in $GPG_KEYS; do \
		gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key"; \
	done; \
	gpg --batch --export "$GPG_KEYS" > /etc/apt/trusted.gpg.d/mariadb.gpg; \
	if command -v gpgconf >/dev/null; then \
		gpgconf --kill all; \
	fi

ENV LANG C.UTF-8

# bashbrew-architectures: amd64 arm64v8 ppc64le s390x
ARG MARIADB_VERSION=1:11.8.6+maria~deb13
ENV MARIADB_VERSION $MARIADB_VERSION
# release-status:Stable
# release-support-type:Long Term Support
# (https://downloads.mariadb.org/rest-api/mariadb/)

RUN set -ex; \
	export PYTHONDONTWRITEBYTECODE=1; \
	\
	dpkgArch="$(dpkg --print-architecture)"; \
	aptRepo="[ signed-by=/etc/apt/trusted.gpg.d/mariadb.gpg ] http://archive.mariadb.org/mariadb-11.8.6/repo/debian/ trixie main"; \
	echo "deb-src $aptRepo" > /etc/apt/sources.list.d/pgdg.list; \
	apt-get update; \
	tempDir="$(mktemp -d)"; \
	cd "$tempDir"; \
	proc="$(nproc)"; \
	export DEB_BUILD_OPTIONS="nocheck parallel=$nproc"; \
	apt-get build-dep -y "mariadb-server=$MARIADB_VERSION"; \
	apt-get source --compile "mariadb-server=$MARIADB_VERSION"; \
	find . -maxdepth 1 -mindepth 1 -type d -exec rm -rf {} +; \
	cd /; \
	mv "$tempDir" /dist

FROM scratch
COPY --from=builder /dist /dist